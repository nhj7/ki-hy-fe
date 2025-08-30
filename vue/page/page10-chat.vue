<template>
    <v-container fluid class="fill-height">
        <v-row justify="center" class="fill-height">
            <v-col cols="12" sm="12" md="10" lg="10">
                <!-- 메인 타이틀 -->
                <v-row v-if="isTitle" justify="center" style="margin-top: 20vh;" class="mb-12">
                    <v-col cols="12" class="text-center">
                        <h1 class="text-h3 text-md-h2 font-weight-regular grey--text text--darken-3">
                            무엇을 도와드릴까요?
                        </h1>
                    </v-col>
                </v-row>

                <!-- 응답 영역 -->
                <v-expand-transition>
                    <div v-if="response && response.length > 0">
                        <v-row v-for="(responseItem, index) in response" :key="index" justify="center" class="mt-3">
                            <v-col cols="12">
                                <v-card elevation="2" rounded="lg">
                                    <v-card-text>
                                        <p class="mb-0 body-1" :class="{ 'loading-text': responseItem.isWaiting }"  v-html="responseItem.msg"></p>
                                    </v-card-text>
                                </v-card>
                            </v-col>
                        </v-row>
                    </div>
                </v-expand-transition>


                <!-- 입력 필드 -->
                <v-row justify="center" class="mt-3">
                    <v-col cols="12">
                        <v-textarea v-model="inputText" placeholder="무엇이든 물어보세요" outlined rounded solo hide-details
                            elevation="2"  @keydown.enter.exact.prevent="handleKeydownEnter" auto-grow rows="1" row-height="24">
                            <template v-slot:prepend-inner>
                                
                            </template>

                            <template v-slot:append>
                                <v-btn color="secondary" small elevation="1" rounded @click="handleSubmit" :disabled="!inputText.trim() || isTyping">
                                    <v-progress-circular v-if="isTyping" indeterminate size="20" width="2" color="white"></v-progress-circular>
                                    <span v-else>전송</span>
                                </v-btn>
                            </template>
                        </v-textarea>

                        <!-- 엔터 입력 안내 멘트 -->
                        <div class="text-center mt-2">
                            <small class="text-caption grey--text">
                                * Shift + Enter 키를 누르면 엔터 입력이 가능해요.
                            </small>
                        </div>

                    </v-col>
                </v-row>

                
            </v-col>
        </v-row>
    </v-container>
</template>
<style scoped>
.loading-text {
    position: relative;
    background: linear-gradient(90deg, 
        #757575 0%, 
        #9e9e9e 25%, 
        #757575 50%, 
        #9e9e9e 75%, 
        #757575 100%);
    background-size: 200% 100%;
    animation: text-shimmer 10s ease-in-out infinite;
    background-clip: text;
    -webkit-background-clip: text;
    color: transparent;
}

@keyframes text-shimmer {
    0% {
        background-position: 200% 0;
    }
    100% {
        background-position: -200% 0;
    }
}

/* 라이트모드용 스타일 */
.theme--light .loading-text {
    background: linear-gradient(90deg, 
        #757575 0%, 
        #b9b7b7 25%, 
        #757575 50%, 
        #bdbcbc 75%, 
        #757575 100%);
    background-size: 200% 100%;
    animation: text-shimmer 10s ease-in-out infinite;
    background-clip: text;
    -webkit-background-clip: text;
    color: transparent;
}

/* 다크모드용 스타일 */
.theme--dark .loading-text {
    background: linear-gradient(90deg, 
        #ecebeb 0%, 
        #c2c1c1 25%, 
        #919191 50%, 
        #bcbaba 75%, 
        #d8d8d8 100%);
    background-size: 200% 100%;
    animation: text-shimmer 10s ease-in-out infinite;
    background-clip: text;
    -webkit-background-clip: text;
    color: transparent;
}
</style>
<script>
const renderer = new marked.Renderer();
renderer.link = function (link) {
    return `<a href="${link.href}" title="${link.text}" target="_blank">${link.text}</a>`;
};
// marked 옵션에 렌더러 적용
marked.setOptions({
    renderer: renderer,
});
module.exports = {
    name: 'MainLayout',
    data() {
        return {
            inputText: '',
            response: [],
            responseIdx : -1,
            isTitle : true,
            isTyping: false,
        }
    },
    computed: {
        micIcon() {
            return this.isMicActive ? 'mdi-microphone' : 'mdi-microphone-outline'
        },
        soundIcon() {
            return this.isSoundActive ? 'mdi-volume-high' : 'mdi-equalizer'
        }
    },
    methods: {
        // sleep 함수 추가
        sleep(ms) {
            return new Promise(resolve => setTimeout(resolve, ms));
        },
        handleKeydownEnter(event) {
            // Enter 키만 눌렀을 때만 처리
            if (event.key === 'Enter' && !event.shiftKey && !event.ctrlKey && !event.altKey && !this.isTyping ) {
                console.log("handleKeydownEnter");
                this.handleSubmit();
            }
        },
        async handleSubmit() {
            let isHTTPS = location.protocol === 'https:';            //isHTTPS = true;
            this.isTitle = false;
            if (this.inputText.trim()) {
                this.responseIdx++;
                // response 배열에 새 객체 추가
                this.response.push( {
                    msg: `"${this.inputText}"에 대한 답변을 준비하고 있어요.`,
                    isWaiting: true
                });
                //this.$loading.show('답변을 준비하고 있습니다...');
                this.isTyping = true; // 전송 시작

                try {
                    
                    const response = await this.$axios.post(`${isHTTPS?'https://cors.iinfo.kr:7403/':''}http://114.207.145.84:8000/chat`, {
                        prompt: this.inputText + ". 한글로 답변해줘."
                    });

                    console.log(response);
                    this.response[this.responseIdx].msg = marked.parse(response.data.response);
                    
                    //await this.sleep(13000);
                    //this.response[this.responseIdx].msg = "답변 완료.";
                    this.inputText = ''
                } catch (error) {
                    //this.$loading.hide();
                    console.error('Error:', error);
                } finally {
                    this.isTyping = false; // 전송 완료
                    this.response[this.responseIdx].isWaiting = false;
                    
                    this.$nextTick(() => {                    
                        // 모든 pre code 블록에 highlight 적용
                        this.$el.querySelectorAll('pre code').forEach((block) => {
                            console.log('block',block);
                            hljs.highlightElement(block);
                        });
                    });
                }
                
            }
        },
        toggleMic() {
            this.isMicActive = !this.isMicActive
            // 음성 인식 로직
        },
        toggleSound() {
            this.isSoundActive = !this.isSoundActive
            // 음성 출력 로직
        }
    }
} 
</script>