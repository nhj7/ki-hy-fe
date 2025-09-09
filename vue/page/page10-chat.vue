<template>
    <v-container fluid class="pa-0" style="height: calc(100vh - 120px);">
        <v-row class="ma-0" style="height: 100%;">
            <!-- 좌측 사이드바 -->
            <v-col cols="3" class="sidebar pa-0">
                <v-card class="fill-height" flat>
                    <!-- 새 채팅 버튼 -->
                    <v-card-title class="pa-3 pb-2">
                        <v-btn color="primary" block outlined @click="startNewChat">
                            <v-icon left>mdi-plus-circle</v-icon>
                            새 채팅
                        </v-btn>
                    </v-card-title>
                    
                    <!-- 최근 대화 섹션 -->
                    <v-card-subtitle class="pa-3 pt-0 pb-2">
                        <v-subheader class="pa-0">최근</v-subheader>
                    </v-card-subtitle>
                    
                    <!-- 채팅 기록 목록 -->
                    <v-card-text class="pa-0 chat-history-list">
                        <v-list dense class="pa-0">
                            <v-list-item 
                                v-for="chat in chatSessions" 
                                :key="chat.id"
                                @click="selectChat(chat.id)"
                                :class="{ 'primary--text': currentChatId === chat.id }"
                                class="chat-item"
                            >
                                <v-list-item-content>
                                    <v-list-item-title class="chat-title">
                                        {{ chat.title }}
                                    </v-list-item-title>
                                    <v-list-item-subtitle class="chat-time">
                                        {{ formatRelativeTime(chat.updatedAt) }}
                                    </v-list-item-subtitle>
                                </v-list-item-content>
                                <v-list-item-action class="chat-actions">
                                    <v-btn icon small @click.stop="deleteChat(chat.id)">
                                        <v-icon small>mdi-delete-outline</v-icon>
                                    </v-btn>
                                </v-list-item-action>
                            </v-list-item>
                        </v-list>
                        
                        <!-- 채팅 기록이 없을 때 -->
                        <div v-if="chatSessions.length === 0" class="pa-3 text-center">
                            <v-icon color="grey" size="48" class="mb-2">mdi-chat-outline</v-icon>
                            <p class="grey--text text-caption">아직 대화 기록이 없습니다</p>
                        </div>
                    </v-card-text>
                </v-card>
            </v-col>
            
            <!-- 메인 채팅 영역 -->
            <v-col cols="9" class="main-chat pa-0" style="height: 100%;">
                <v-card class="fill-height d-flex flex-column" flat>
                    <!-- 채팅 응답 영역 -->
                    <v-card-text class="flex-grow-1 chat-messages pa-3" style="overflow-y: auto;">
                        <!-- 메인 타이틀 (첫 화면) -->
                        <div v-if="isTitle" class="text-center" style="margin-top: 30vh;">
                            <h1 class="text-h4 text-md-h4 font-weight-regular grey--text text--darken-3">
                                무엇을 도와드릴까요?
                            </h1>
                        </div>

                        <!-- 채팅 메시지 영역 -->
                        <div v-if="currentMessages && currentMessages.length > 0" class="chat-messages-content">
                            <div v-for="(message, index) in currentMessages" :key="message.id || index" class="mb-4">
                                <!-- 사용자 질문 -->
                                <div v-if="message.type === 'user'" class="user-message mb-3">
                                    <div class="d-flex justify-end">
                                        <v-card elevation="1" rounded="lg" color="primary" dark class="user-card">
                                            <v-card-text class="pa-3">
                                                <p class="mb-0 body-1">{{ message.content }}</p>
                                            </v-card-text>
                                        </v-card>
                                    </div>
                                </div>
                                
                                <!-- AI 응답 -->
                                <div v-if="message.type === 'assistant'" class="assistant-message">
                                    <div class="d-flex justify-start">
                                        <v-card elevation="2" rounded="lg" class="assistant-card">
                                            <v-card-text class="pa-3">
                                                <p class="mb-0 body-1" :class="{ 'loading-text': message.isWaiting }" v-html="message.content"></p>
                                            </v-card-text>
                                        </v-card>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </v-card-text>

                    <!-- 입력 필드 -->
                    <v-card-actions class="pa-3 pt-0 justify-center">
                        <div style="width: 80%;">
                            <v-textarea 
                                v-model="inputText" 
                                placeholder="무엇이든 물어보세요" 
                                outlined 
                                rounded 
                                solo 
                                hide-details
                                elevation="2"  
                                @keydown.enter.exact.prevent="handleKeydownEnter" 
                                auto-grow 
                                rows="1" 
                                row-height="24"
                            >
                                <template v-slot:append>
                                    <v-btn color="secondary" small elevation="1" rounded @click="handleSubmit" :disabled="!inputText.trim() || isTyping">
                                        <v-progress-circular v-if="isTyping" indeterminate size="20" width="2" color="white"></v-progress-circular>
                                        <span v-else>전송</span>
                                    </v-btn>
                                </template>
                            </v-textarea>
                        </div>
                    </v-card-actions>
                    
                    <!-- 엔터 입력 안내 멘트 -->
                    <div class="text-center pb-3">
                        <small class="text-caption grey--text">
                            * Shift + Enter 키를 누르면 엔터 입력이 가능해요.
                        </small>
                    </div>
                </v-card>
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

/* 사이드바 스타일 */
.sidebar {
    /* border-right: 1px solid #e0e0e0;
    background-color: #fafafa; */
}

.theme--dark .sidebar {
    border-right: 1px solid #616161;
    background-color: #1e1e1e;
}

/* 채팅 기록 목록 */
.chat-history-list {
    height: calc(100vh - 300px);
    overflow-y: auto;
    font-size: 0.9rem !important;
}

.chat-item {
    border-radius: 8px;    
    transition: background-color 0.2s;
    padding : 0 8px !important;
}

.chat-item:hover {
    background-color: rgba(0, 0, 0, 0.04);
}

.theme--dark .chat-item:hover {
    background-color: rgba(255, 255, 255, 0.08);
}

.chat-title {
    font-size: 0.9rem !important;
    line-height: 1.2 !important;
    word-wrap: break-word;
    overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
}

.chat-time {
    font-size: 0.75rem !important;
    opacity: 0.7;
}

.chat-actions {
    opacity: 0;
    transition: opacity 0.2s;
}

.chat-item:hover .chat-actions {
    opacity: 1;
}

/* 메인 채팅 영역 */
.main-chat {
    overflow: hidden;
}

.chat-messages {
    height: calc(100vh - 250px);
    overflow-y: auto;
}

/* 사용자 메시지 스타일 */
.user-card {
    max-width: 70%;
    margin-left: 30%;
}

/* AI 응답 메시지 스타일 */
.assistant-card {
    max-width: 85%;
    margin-right: 15%;
}

.user-message .v-card {
    background-color: var(--v-primary-base) !important;
}

.assistant-message .v-card {
    background-color: var(--v-surface-base);
}

.theme--dark .assistant-message .v-card {
    background-color: var(--v-surface-lighten1);
}

/* 코드 블록 컨테이너 스타일 */
.code-block-container {
    position: relative;
    margin: 16px 0;
    border-radius: 8px;
    overflow: hidden;
    background-color: #f5f5f5;
}

.theme--dark .code-block-container {
    background-color: #2d2d2d;
}

.code-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 8px 12px;
    background-color: #e8e8e8;
    border-bottom: 1px solid #ddd;
    font-size: 0.85rem;
}

.theme--dark .code-header {
    background-color: #404040;
    border-bottom: 1px solid #555;
}

.language-tag {
    font-weight: 500;
    color: #666;
}

.theme--dark .language-tag {
    color: #ccc;
}

.copy-btn {
    display: flex;
    align-items: center;
    gap: 4px;
    background: none;
    border: none;
    color: #666;
    cursor: pointer;
    font-size: 0.8rem;
    padding: 4px 8px;
    border-radius: 4px;
    transition: all 0.2s;
}

.copy-btn:hover {
    background-color: rgba(0, 0, 0, 0.1);
    color: #333;
}

.theme--dark .copy-btn {
    color: #ccc;
}

.theme--dark .copy-btn:hover {
    background-color: rgba(255, 255, 255, 0.1);
    color: #fff;
}

.copy-text {
    font-size: 0.75rem;
}

.code-block-container pre {
    margin: 0;
    padding: 12px;
    background-color: transparent;
    border-radius: 0;
}

.code-block-container code {
    background-color: transparent;
    padding: 0;
}
</style>
<script>
const renderer = new marked.Renderer();
renderer.link = function (link) {
    return `<a href="${link.href}" title="${link.text}" target="_blank">${link.text}</a>`;
};

renderer.code = function(code, language) {
    console.log('=== 코드 렌더러 디버깅 ===');
    console.log('code type:', typeof code);
    console.log('code value:', code);
    console.log('language type:', typeof language);
    console.log('language value:', language);
    console.log('arguments length:', arguments.length);
    console.log('all arguments:', arguments);
    
    // 안전한 값 추출
    let codeText = '';
    let langText = '';
    
    if (typeof code === 'string') {
        codeText = code;
        langText = language || 'code';
    } else if (code && typeof code === 'object') {
        codeText = code.text || code.code || code.raw || '';
        langText = code.lang || code.language || language || 'code';
    }
    
    console.log('최종 codeText:', codeText);
    console.log('최종 langText:', langText);
    
    const langClass = langText ? `language-${langText}` : '';
    const langDisplay = langText || 'code';
    
    return `
        <div class="code-block-container">
            <div class="code-header">
                <span class="language-tag">${langDisplay}</span>
                <button class="copy-btn" type="button">
                    <i class="mdi mdi-content-copy"></i>
                    <span class="copy-text">복사</span>
                </button>
            </div>
            <pre><code class="${langClass}">${codeText}</code></pre>
        </div>
    `;
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
            currentChatId: null,
            chatSessions: [],
            db: null,
            currentMessages: [],
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
            let isHTTPS = location.protocol === 'https:';
            this.isTitle = false;
            const postFix = "한글로 답변해줘.";
            if (this.inputText.trim()) {
                const userQuestion = this.inputText;
                
                // 새 채팅 세션 생성 (첫 질문일 때)
                if (!this.currentChatId) {
                    await this.createNewChatSession();
                }
                
                // 사용자 질문을 currentMessages와 IndexedDB에 추가
                const userMessage = {
                    id: this.$util.uuid(),
                    type: 'user',
                    content: userQuestion,
                    timestamp: Date.now(),
                    isWaiting: false
                };
                
                this.currentMessages.push(userMessage);
                await this.addMessageToCurrentSession(userMessage);
                
                // 대기 중인 AI 응답 메시지 추가
                const waitingMessage = {
                    id: this.$util.uuid(),
                    type: 'assistant',
                    content: `"${userQuestion}"에 대한 답변을 준비하고 있어요.`,
                    timestamp: Date.now(),
                    isWaiting: true
                };
                
                this.currentMessages.push(waitingMessage);
                this.isTyping = true;
                this.inputText = '';
                
                // 스크롤을 하단으로 이동
                this.scrollToBottom();

                try {
                    const response = await this.$axios.post(`${isHTTPS?'https://cors.iinfo.kr:7403/':''}http://114.207.145.84:8000/chat`, {
                        prompt: userQuestion + ". " + postFix
                    });

                    console.log(response);
                    let assistantResponse = response.data.response;
                    
                    // 응답에서 질문 부분 제거
                    const questionPrefix = `${userQuestion}. ${postFix}`;
                    if (assistantResponse.startsWith(questionPrefix)) {
                        assistantResponse = assistantResponse.substring(questionPrefix.length).trim();
                    }
                    
                    const assistantContent = marked.parse(assistantResponse);
                    
                    // 대기 메시지를 실제 응답으로 교체
                    const waitingIndex = this.currentMessages.findIndex(msg => msg.id === waitingMessage.id);
                    if (waitingIndex !== -1) {
                        this.currentMessages[waitingIndex].content = assistantContent;
                        this.currentMessages[waitingIndex].isWaiting = false;
                    }
                    
                    // IndexedDB에 AI 응답 저장
                    await this.addMessageToCurrentSession({
                        id: waitingMessage.id,
                        type: 'assistant',
                        content: assistantContent,
                        timestamp: Date.now(),
                        isWaiting: false
                    });
                    
                } catch (error) {
                    console.error('Error:', error);
                } finally {
                    this.isTyping = false;
                    
                    this.$nextTick(() => {
                        this.setupCodeBlocks();
                        this.scrollToBottom();
                    });
                }
            }
        },
        
        // IndexedDB 초기화
        async initDB() {
            return new Promise((resolve, reject) => {
                const request = indexedDB.open('KIChatDB', 1);
                
                request.onerror = () => reject(request.error);
                request.onsuccess = () => {
                    this.db = request.result;
                    resolve(request.result);
                };
                
                request.onupgradeneeded = (event) => {
                    const db = event.target.result;
                    
                    // 채팅 세션 스토어 생성
                    if (!db.objectStoreNames.contains('chatSessions')) {
                        const store = db.createObjectStore('chatSessions', { keyPath: 'id' });
                        store.createIndex('createdAt', 'createdAt', { unique: false });
                        store.createIndex('updatedAt', 'updatedAt', { unique: false });
                    }
                };
            });
        },

        // 새 채팅 세션 생성
        async createNewChatSession() {
            this.currentChatId = this.$util.uuid();
            const newSession = {
                id: this.currentChatId,
                title: this.generateChatTitle(this.inputText),
                firstQuestion: this.inputText,
                messages: [],
                createdAt: Date.now(),
                updatedAt: Date.now(),
                messageCount: 0
            };
            
            this.chatSessions.unshift(newSession);
            this.currentMessages = [];
            await this.saveChatSession(newSession);
            console.log('새 채팅 세션 생성:', newSession);
        },

        // 채팅 세션 저장
        async saveChatSession(session) {
            if (!this.db) await this.initDB();
            
            return new Promise((resolve, reject) => {
                const transaction = this.db.transaction(['chatSessions'], 'readwrite');
                const store = transaction.objectStore('chatSessions');
                
                const request = store.put(session);
                request.onsuccess = () => resolve(request.result);
                request.onerror = () => reject(request.error);
            });
        },

        // 모든 채팅 세션 불러오기
        async loadAllChatSessions() {
            if (!this.db) await this.initDB();
            
            return new Promise((resolve, reject) => {
                const transaction = this.db.transaction(['chatSessions'], 'readonly');
                const store = transaction.objectStore('chatSessions');
                const index = store.index('updatedAt');
                
                const request = index.getAll();
                request.onsuccess = () => {
                    // 최신순으로 정렬
                    const sessions = request.result.sort((a, b) => b.updatedAt - a.updatedAt);
                    resolve(sessions);
                };
                request.onerror = () => reject(request.error);
            });
        },

        // 특정 채팅 세션 불러오기
        async loadChatSession(chatId) {
            if (!this.db) await this.initDB();
            
            return new Promise((resolve, reject) => {
                const transaction = this.db.transaction(['chatSessions'], 'readonly');
                const store = transaction.objectStore('chatSessions');
                
                const request = store.get(chatId);
                request.onsuccess = () => resolve(request.result);
                request.onerror = () => reject(request.error);
            });
        },

        // 채팅 세션 삭제
        async deleteChatSession(chatId) {
            if (!this.db) await this.initDB();
            
            return new Promise((resolve, reject) => {
                const transaction = this.db.transaction(['chatSessions'], 'readwrite');
                const store = transaction.objectStore('chatSessions');
                
                const request = store.delete(chatId);
                request.onsuccess = () => resolve(request.result);
                request.onerror = () => reject(request.error);
            });
        },

        // 현재 채팅 세션에 메시지 추가
        async addMessageToCurrentSession(message) {
            if (!this.currentChatId) return;
            
            const currentSession = this.chatSessions.find(s => s.id === this.currentChatId);
            if (currentSession) {
                currentSession.messages.push(message);
                currentSession.updatedAt = Date.now();
                currentSession.messageCount = currentSession.messages.length;
                
                await this.saveChatSession(currentSession);
            }
        },
        
        // 채팅 제목 생성 (30자 제한)
        generateChatTitle(question) {
            const maxLength = 30;
            if (question.length <= maxLength) {
                return question;
            }
            return question.substring(0, maxLength) + '...';
        },
        
        // 새 채팅 시작
        startNewChat() {
            this.currentChatId = null;
            this.response = [];
            this.responseIdx = -1;
            this.isTitle = true;
            this.inputText = '';
            this.currentMessages = [];
        },
        
        // 채팅 선택
        async selectChat(chatId) {
            this.currentChatId = chatId;
            this.isTitle = false;
            
            try {
                const session = await this.loadChatSession(chatId);
                if (session && session.messages) {
                    // 전체 메시지 히스토리 로드
                    this.currentMessages = [...session.messages];
                    
                    // 코드 하이라이트 적용
                    this.$nextTick(() => {
                        this.setupCodeBlocks();
                        this.scrollToBottom();
                    });
                }
                console.log('채팅 선택:', session);
            } catch (error) {
                console.error('채팅 로드 오류:', error);
            }
        },
        
        // 채팅 삭제
        async deleteChat(chatId) {
            try {
                await this.deleteChatSession(chatId);
                const index = this.chatSessions.findIndex(chat => chat.id === chatId);
                if (index !== -1) {
                    this.chatSessions.splice(index, 1);
                }
                if (this.currentChatId === chatId) {
                    this.startNewChat();
                }
                console.log('채팅 삭제 완료:', chatId);
            } catch (error) {
                console.error('채팅 삭제 오류:', error);
            }
        },
        
        // 상대적 시간 포맷
        formatRelativeTime(timestamp) {
            const now = Date.now();
            const diff = now - timestamp;
            const minutes = Math.floor(diff / (1000 * 60));
            const hours = Math.floor(diff / (1000 * 60 * 60));
            const days = Math.floor(diff / (1000 * 60 * 60 * 24));
            
            if (minutes < 1) return '방금 전';
            if (minutes < 60) return `${minutes}분 전`;
            if (hours < 24) return `${hours}시간 전`;
            if (days < 7) return `${days}일 전`;
            return new Date(timestamp).toLocaleDateString();
        },

        // 채팅 영역을 하단으로 스크롤
        scrollToBottom() {
            this.$nextTick(() => {
                const chatMessagesElement = this.$el.querySelector('.chat-messages');
                if (chatMessagesElement) {
                    chatMessagesElement.scrollTop = chatMessagesElement.scrollHeight;
                }
            });
        },

        // 코드 블록 설정 (하이라이트 + 복사 버튼)
        setupCodeBlocks() {
            // 기존 이벤트 리스너 제거
            this.$el.querySelectorAll('.copy-btn').forEach(btn => {
                btn.removeEventListener('click', this.handleCopyCode);
            });
            
            // 코드 하이라이트 적용
            this.$el.querySelectorAll('pre code').forEach((block) => {
                hljs.highlightElement(block);
            });
            
            // 복사 버튼에 이벤트 리스너 추가
            this.$el.querySelectorAll('.copy-btn').forEach(btn => {
                btn.addEventListener('click', this.handleCopyCode);
            });
        },

        // 코드 복사 처리
        handleCopyCode(event) {
            const button = event.currentTarget;
            const codeContainer = button.closest('.code-block-container');
            const codeElement = codeContainer.querySelector('code');
            const codeText = codeElement.textContent || codeElement.innerText;
            
            navigator.clipboard.writeText(codeText).then(() => {
                // 복사 성공 피드백
                const originalHTML = button.innerHTML;
                button.innerHTML = '<i class="mdi mdi-check"></i><span class="copy-text">복사됨</span>';
                button.style.color = '#4CAF50';
                
                setTimeout(() => {
                    button.innerHTML = originalHTML;
                    button.style.color = '';
                }, 2000);
                
            }).catch(err => {
                console.error('복사 실패:', err);
                if (this.$msg) {
                    this.$msg.showSnackbar('복사에 실패했습니다', 'error', 2000);
                }
            });
        },
        toggleMic() {
            this.isMicActive = !this.isMicActive
            // 음성 인식 로직
        },
        toggleSound() {
            this.isSoundActive = !this.isSoundActive
            // 음성 출력 로직
        }
    },
    
    async mounted() {
        try {
            // IndexedDB 초기화
            await this.initDB();
            
            // 기존 채팅 세션 로드
            const sessions = await this.loadAllChatSessions();
            this.chatSessions = sessions;
            
            console.log('채팅 세션 로드 완료:', sessions.length, '개');
        } catch (error) {
            console.error('컴포넌트 초기화 오류:', error);
        }
    },
    
    beforeDestroy() {
        // IndexedDB 연결 정리
        if (this.db) {
            this.db.close();
        }
    }
} 
</script>