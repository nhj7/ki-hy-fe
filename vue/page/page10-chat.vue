<template>
    <v-container fluid class="fill-height">
        <v-row justify="center" class="fill-height">
            <v-col cols="12" sm="10" md="8" lg="6">
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
                    <v-row v-if="response" justify="center" class="mt-6">
                        <v-col cols="12">
                            <v-card elevation="2" rounded="lg">
                                <v-card-text>
                                    <p class="mb-0 body-1" v-html="response"></p>
                                </v-card-text>
                            </v-card>
                        </v-col>
                    </v-row>
                </v-expand-transition>


                <!-- 입력 필드 -->
                <v-row justify="center">
                    <v-col cols="12">
                        <v-textarea v-model="inputText" placeholder="무엇이든 물어보세요" outlined rounded solo hide-details
                            elevation="2" @keydown.enter.prevent="handleSubmit" auto-grow rows="1" row-height="24">
                            <template v-slot:prepend-inner>
                                
                            </template>

                            <template v-slot:append>
                                <v-btn color="secondary" elevation="1" rounded @click="handleSubmit" :disabled="!inputText.trim()">
                                    전송
                                </v-btn>
                            </template>
                        </v-textarea>
                    </v-col>
                </v-row>

                
            </v-col>
        </v-row>
    </v-container>
</template>

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
            response: '',
            isTitle : true,
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
        async handleSubmit() {
            this.isTitle = false;
            if (this.inputText.trim()) {
                this.response = `"${this.inputText}"에 대한 답변을 준비하고 있습니다...`
                this.$loading.show('답변을 준비하고 있습니다...');
                const response = await this.$axios.post('http://114.207.145.84:8000/chat', {
                    prompt: this.inputText + ". 한글로 답변해줘."
                });

                console.log(response);

                this.response = marked.parse(response.data.response);
                this.$nextTick(() => {                    
                    // 모든 pre code 블록에 highlight 적용
                    this.$el.querySelectorAll('pre code').forEach((block) => {
                        console.log('block',block);
                        hljs.highlightElement(block);
                    });
                });
                this.inputText = ''

                this.$loading.hide();
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