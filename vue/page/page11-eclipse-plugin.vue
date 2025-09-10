<template>
    <v-container fluid class="fill-height">
        <v-row justify="center" class="fill-height">
            <v-col cols="12" sm="12" md="10" lg="10">
                <!-- 메인 타이틀 -->
                <v-row justify="center" class="mb-8">
                    <v-col cols="12" class="text-center">
                        <v-icon size="64" color="primary" class="mb-4">mdi-eclipse</v-icon>
                        <h1 class="text-h3 text-md-h2 font-weight-regular grey--text text--darken-3">
                            KI Assist Eclipse Plugin
                        </h1>
                        <p class="text-h6 grey--text text--darken-1 mt-2">
                            이클립스에서 KI Assist 기능을 사용할 수 있는 플러그인입니다
                        </p>
                    </v-col>
                </v-row>

                <!-- 플러그인 정보 카드 -->
                <v-row justify="center" class="mb-6">
                    <v-col cols="12" md="8">
                        <v-card elevation="3" rounded="lg">
                            <v-card-title class="text-h5 primary--text">
                                <v-icon left color="primary">mdi-information</v-icon>
                                플러그인 정보
                            </v-card-title>
                            <v-card-text>
                                <v-list dense>
                                    <v-list-item>
                                        <v-list-item-icon>
                                            <v-icon color="primary">mdi-package-variant</v-icon>
                                        </v-list-item-icon>
                                        <v-list-item-content>
                                            <v-list-item-title><strong>버전:</strong> 1.0.0</v-list-item-title>
                                        </v-list-item-content>
                                    </v-list-item>
                                    <v-list-item>
                                        <v-list-item-icon>
                                            <v-icon color="primary">mdi-calendar</v-icon>
                                        </v-list-item-icon>
                                        <v-list-item-content>
                                            <v-list-item-title><strong>출시일:</strong> 2025년 9월</v-list-item-title>
                                        </v-list-item-content>
                                    </v-list-item>
                                    <v-list-item>
                                        <v-list-item-icon>
                                            <v-icon color="primary">mdi-application-settings</v-icon>
                                        </v-list-item-icon>
                                        <v-list-item-content>
                                            <v-list-item-title><strong>지원 버전:</strong> Eclipse Indigo 2011-06 이상</v-list-item-title>
                                        </v-list-item-content>
                                    </v-list-item>
                                    <v-list-item>
                                        <v-list-item-icon>
                                            <v-icon color="primary">mdi-folder-zip</v-icon>
                                        </v-list-item-icon>
                                        <v-list-item-content>
                                            <v-list-item-title><strong>파일 크기:</strong> 약 1.MB 이하</v-list-item-title>
                                        </v-list-item-content>
                                    </v-list-item>
                                </v-list>
                            </v-card-text>
                        </v-card>
                    </v-col>
                </v-row>

                <!-- 다운로드 섹션 -->
                <v-row justify="center" class="mb-6">
                    <v-col cols="12" md="8">
                        <v-card elevation="3" rounded="lg" class="text-center pa-6">
                            <v-icon size="48" color="success" class="mb-4">mdi-download</v-icon>
                            <h2 class="text-h4 mb-4">플러그인 다운로드</h2>
                            <p class="text-body-1 mb-6">
                                아래 버튼을 클릭하여 KI Assist Eclipse Plugin을 다운로드하세요.
                            </p>
                            <v-btn 
                                color="primary" 
                                size="x-large" 
                                elevation="2" 
                                rounded 
                                @click="downloadPlugin"
                                :loading="isDownloading"
                                :disabled="isDownloading"
                            >
                                <v-icon left>mdi-download</v-icon>
                                {{ isDownloading ? '다운로드 중...' : 'KI Assist Plugin 다운로드' }}
                            </v-btn>
                            <p class="text-caption grey--text mt-3">
                                * ZIP 파일로 압축되어 있습니다
                            </p>
                        </v-card>
                    </v-col>
                </v-row>

                <!-- 설치 방법 안내 -->
                <v-row justify="center" class="mb-6">
                    <v-col cols="12" md="8">
                        <v-card elevation="3" rounded="lg">
                            <v-card-title class="text-h5 primary--text">
                                <v-icon left>mdi-install</v-icon>
                                설치 방법
                            </v-card-title>
                            <v-card-text>
                                <v-stepper v-model="currentStep" vertical>
                                    <v-stepper-step :complete="currentStep > 1" step="1">
                                        다운로드 및 압축 해제
                                        <small>ZIP 파일을 다운로드하고 압축을 해제합니다</small>
                                    </v-stepper-step>

                                    <v-stepper-content step="1">
                                        <v-card class="mb-4" elevation="2">
                                            <v-card-text>
                                                <ol class="text-body-1">
                                                    <li>위의 다운로드 버튼을 클릭하여 ZIP 파일을 다운로드합니다</li>
                                                    <li>다운로드된 ZIP 파일을 원하는 위치에 압축 해제합니다</li>
                                                    <li>압축 해제된 폴더에 <code>plugins</code> 폴더가 있는지 확인합니다</li>
                                                </ol>
                                            </v-card-text>
                                        </v-card>
                                        <v-btn color="primary" @click="currentStep = 2">다음</v-btn>
                                    </v-stepper-content>

                                    <v-stepper-step :complete="currentStep > 2" step="2">
                                        이클립스에 플러그인 설치
                                        <small>압축 해제된 플러그인을 이클립스에 설치합니다</small>
                                    </v-stepper-step>

                                    <v-stepper-content step="2">
                                        <v-card class="mb-4" elevation="2">
                                            <v-card-text>
                                                <ol class="text-body-1">
                                                    <li>이클립스를 실행합니다</li>
                                                    <li><strong>Help</strong> → <strong>Install New Software...</strong>를 선택합니다</li>
                                                    <li><strong>Add...</strong> 버튼을 클릭합니다</li>
                                                    <li><strong>Name</strong>에 "KI Assist"를 입력합니다</li>
                                                    <li><strong>Location</strong>에 압축 해제된 폴더 경로를 입력하거나 <strong>Archive...</strong>를 클릭하여 ZIP 파일을 직접 선택합니다</li>
                                                    <li><strong>OK</strong>를 클릭합니다</li>
                                                </ol>
                                            </v-card-text>
                                        </v-card>
                                        <v-btn text @click="currentStep = 1">이전</v-btn>
                                        <v-btn color="primary" @click="currentStep = 3">다음</v-btn>
                                    </v-stepper-content>

                                    <v-stepper-step :complete="currentStep > 3" step="3">
                                        설치 완료 및 재시작
                                        <small>플러그인 설치를 완료하고 이클립스를 재시작합니다</small>
                                    </v-stepper-step>

                                    <v-stepper-content step="3">
                                        <v-card class="mb-4" elevation="2">
                                            <v-card-text>
                                                <ol class="text-body-1">
                                                    <li>설치할 플러그인 목록에서 "KI Assist"를 체크합니다</li>
                                                    <li><strong>Next</strong>를 클릭하여 라이선스 동의 및 설치를 진행합니다</li>
                                                    <li>설치가 완료되면 이클립스를 재시작합니다</li>
                                                    <li>재시작 후 <strong>Window</strong> → <strong>Perspective</strong> → <strong>Open Perspective</strong> → <strong>Other...</strong>에서 "KI Assist"를 찾을 수 있습니다</li>
                                                </ol>
                                            </v-card-text>
                                        </v-card>
                                        <v-btn text @click="currentStep = 2">이전</v-btn>
                                        <v-btn color="success" @click="currentStep = 1">완료</v-btn>
                                    </v-stepper-content>
                                </v-stepper>
                            </v-card-text>
                        </v-card>
                    </v-col>
                </v-row>

                <!-- 주요 기능 안내 >
                <v-row justify="center" class="mb-6">
                    <v-col cols="12" md="8">
                        <v-card elevation="3" rounded="lg">
                            <v-card-title class="text-h5 primary--text">
                                <v-icon left>mdi-star</v-icon>
                                주요 기능
                            </v-card-title>
                            <v-card-text>
                                <v-row>
                                    <v-col cols="12" md="6">
                                        <v-list dense>
                                            <v-list-item>
                                                <v-list-item-icon>
                                                    <v-icon color="success">mdi-check</v-icon>
                                                </v-list-item-icon>
                                                <v-list-item-content>
                                                    <v-list-item-title>코드 자동 완성</v-list-item-title>
                                                </v-list-item-content>
                                            </v-list-item>
                                            <v-list-item>
                                                <v-list-item-icon>
                                                    <v-icon color="success">mdi-check</v-icon>
                                                </v-list-item-icon>
                                                <v-list-item-content>
                                                    <v-list-item-title>실시간 코드 분석</v-list-item-title>
                                                </v-list-item-content>
                                            </v-list-item>
                                        </v-list>
                                    </v-col>
                                    <v-col cols="12" md="6">
                                        <v-list dense>
                                            <v-list-item>
                                                <v-list-item-icon>
                                                    <v-icon color="success">mdi-check</v-icon>
                                                </v-list-item-icon>
                                                <v-list-item-content>
                                                    <v-list-item-title>스마트 리팩토링</v-list-item-title>
                                                </v-list-item-content>
                                            </v-list-item>
                                            <v-list-item>
                                                <v-list-item-icon>
                                                    <v-icon color="success">mdi-check</v-icon>
                                                </v-list-item-icon>
                                                <v-list-item-content>
                                                    <v-list-item-title>통합 디버깅</v-list-item-title>
                                                </v-list-item-content>
                                            </v-list-item>
                                        </v-list>
                                    </v-col>
                                </v-row>
                            </v-card-text>
                        </v-card>
                    </v-col>
                </v-row-->

                <!-- 문의 및 지원 -->
                <v-row justify="center">
                    <v-col cols="12" md="8">
                        <v-card elevation="3" rounded="lg" class="text-center pa-6">
                            <v-icon size="48" color="info" class="mb-4">mdi-help-circle</v-icon>
                            <h3 class="text-h5 mb-4">도움이 필요하신가요?</h3>
                            <p class="text-body-1 mb-4">
                                설치 과정에서 문제가 발생하거나 추가 지원이 필요한 경우 연락해주세요.
                            </p>
                            
                            <!-- 연락처 정보 -->
                            <v-row justify="center" class="mb-0">
                                <v-col cols="12" md="6" class="mb-0">
                                    <v-card outlined class="pa-3 d-flex flex-column" style="height: 160px;">
                                        <div class="text-center">
                                            <v-icon color="primary" class="mb-2">mdi-phone</v-icon>
                                            <div class="text-h6 font-weight-bold primary--text">031-789-3996</div>
                                            <div class="text-caption grey--text">전화 문의</div>
                                        </div>
                                        <v-spacer></v-spacer>
                                        <div class="mt-3">
                                            <v-btn color="info" outlined @click="callSupport" block>
                                                <v-icon left>mdi-phone</v-icon>
                                                전화 문의
                                            </v-btn>
                                        </div>
                                    </v-card>
                                </v-col>
                                <v-col cols="12" md="6" class="mb-0">
                                    <v-card outlined class="pa-3 d-flex flex-column" style="height: 160px;">
                                        <div class="text-center">
                                            <v-icon color="primary" class="mb-2">mdi-email</v-icon>
                                            <div class="text-body-1 font-weight-bold primary--text">S11971@koreainvestment.com</div>
                                            <div class="text-caption grey--text">이메일 문의</div>
                                        </div>
                                        <v-spacer></v-spacer>
                                        <div class="mt-3">
                                            <v-btn color="primary"  outlined @click="contactSupport" block>
                                                <v-icon left>mdi-email</v-icon>
                                                이메일 문의
                                            </v-btn>
                                        </div>
                                    </v-card>
                                </v-col>
                            </v-row>
                        </v-card>
                    </v-col>
                </v-row>
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
module.exports = {
    name: 'EclipsePluginPage',
    data() {
        return {
            currentStep: 1,
            isDownloading: false,
        }
    },
    methods: {
        async downloadPlugin() {
            this.isDownloading = true;
            
            try {
                // 실제 다운로드 로직 구현
                // 예시: ZIP 파일 다운로드
                const downloadUrl = '/downloads/ki-assist-eclipse-plugin.zip';
                
                // 브라우저 다운로드 트리거
                const link = document.createElement('a');
                link.href = downloadUrl;
                link.download = 'ki-assist-eclipse-plugin.zip';
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
                
                // 성공 메시지
                this.$toast.success('플러그인 다운로드가 시작되었습니다.');
                
            } catch (error) {
                console.error('Download error:', error);
                this.$toast.error('다운로드 중 오류가 발생했습니다.');
            } finally {
                this.isDownloading = false;
            }
        },
        
        contactSupport() {
            // 지원팀 문의 로직 구현
            // 예: 이메일 클라이언트 열기 또는 문의 폼으로 이동
            window.open('mailto:S11971@koreainvestment.com?subject=KI Assist Eclipse Plugin 지원 요청', '_blank');
        },

        callSupport() {
            // 전화 문의 로직 구현
            // 예: 전화 걸기 또는 전화 번호 클립보드에 복사
            const phoneNumber = '031-789-3996';
            if (navigator.userAgent.includes('Macintosh')) {
                window.open(`tel:${phoneNumber}`, '_self');
            } else {
                window.open(`tel:${phoneNumber}`, '_self');
            }
        }
    }
}
</script>

<style scoped>
.v-stepper {
    box-shadow: none;
}

.v-stepper__content {
    padding: 16px 0;
}

code {
    background-color: #f5f5f5;
    padding: 2px 6px;
    border-radius: 4px;
    font-family: 'Courier New', monospace;
}

.theme--dark code {
    background-color: #424242;
}
</style>
