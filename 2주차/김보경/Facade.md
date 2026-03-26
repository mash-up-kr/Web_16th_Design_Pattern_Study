# 퍼사드 패턴
- 사용자는 내부의 복잡성을 몰라도 되는 패턴
- 복잡한 시스템을 감싸서, 간단한 인터페이스만 제공하는것이 퍼사드 패턴의 핵심 입니다.
- 개인적으로 HAL 혹은 API가 아주 유명한 퍼사드 패턴이지 않을까 생각합니다.
  - HAL: 하드웨어 추상화 계층(Hardware Abstraction Layer)이란, 하드웨어의 구체적인 특징 및 복잡한 내부 구조를 감추고, 깔끔하고 일관성 있는 인터페이스를 제공한다. 기반이 되는 하드웨어의 종류에 관계없이 단순, portable하고 추상화된 API를 통해 응용 프로그램들이 호스트 시스템의 컴퓨터 하드웨어를 발견하고 사용할 수 있게 하는 것이다.
  - API: API(Application Programming Interface, 응용 프로그램 프로그래밍 인터페이스)는 응용 프로그램에서 사용할 수 있도록, 운영 체제나 프로그래밍 언어가 제공하는 기능을 제어할 수 있게 만든 인터페이스를 뜻한다. 주로 파일 제어, 창 제어, 화상 처리, 문자 제어 등을 위한 인터페이스를 제공한다.

- 퍼사드 스타일 커스텀 훅
  ```tsx
  // 복잡한 API 호출 + 상태 관리 로직을 퍼사드 훅으로 감싸기
  const useUserDashboard = () => {
    const { data: user } = useFetchUser();
    const { data: orders } = useFetchOrders();
    const { data: notifications } = useFetchNotifications();

    return { user, orders, notifications }; // 단순화된 인터페이스
  };

  // 컴포넌트에서는 퍼사드 훅만 사용
  const Dashboard = () => {
    const { user, orders } = useUserDashboard(); // 복잡성은 퍼사드 뒤에
    return <>{/* 렌더링 */}</>;
  };
  ```
  - 비즈니스 로직을 담고있는 훅들을 사용처에서 필요한 것만 추출하여 사용할 수 있도록 단순화

- 퍼사드 스타일 Next.js 인증
  ```ts
  // pages/api/auth.js
  import { authWithGoogle, authWithEmail, handleTokenRefresh } from '@lib/auth';

  // 복잡한 인증 로직을 단일 엔드포인트로 추상화
  export default async function handler(req, res) {
    if (req.method === 'POST') {
      const { provider } = req.body;
      let result;

      if (provider === 'google') result = await authWithGoogle(req.body);
      else if (provider === 'email') result = await authWithEmail(req.body);

      res.status(200).json(result); // 클라이언트는 통일된 응답 받음
    }
  }
  ```
  - 복잡성을 감추는것이 핵심입니다. 개별 로직의 구조를 알 필요 없이 필요한 내용만 조합해서 간단하게 사용 가능한 구조를 구성

## Axios + 캐싱 + 에러 처리 통합 예시
- 아주 오랫동안 사용되어왔언 유우명한 대표적인 사용 예시라고 합니다.
  - Axios로 HTTP 요청
  - 자동 캐싱 (같은 URL 요청 시 중복 호출 방지)
  - 통합 에러 처리 (네트워크 오류, 서버 오류, 타임아웃 등)
  ```ts
  // src/libs/httpClient.ts
  import axios, { AxiosInstance, AxiosRequestConfig, AxiosError } from 'axios';

  type CacheKey = string;

  class HttpClient {
    /** 인스턴스 생성 */
    private axiosInstance: AxiosInstance;
    private cache: Map<CacheKey, Promise<any>> = new Map();

    constructor(baseURL: string) {
      this.axiosInstance = axios.create({
        baseURL,
        timeout: 5000,
      });

      this.setupInterceptors();
    }
    // 인스턴스 생성 이후 여러가지 전략을 설정 진행

    private generateCacheKey(config: AxiosRequestConfig): CacheKey {
      return `${config.method}-${config.url}-${JSON.stringify(config.params)}`;
    }

    private setupInterceptors() {
      // 요청 캐싱 인터셉터
      // 왜 req에서 캐싱을 처리하는가?
      // 이미 캐싱이 되어있으면 캐싱된 내용을 반환만 해주면 됨
      this.axiosInstance.interceptors.request.use((config) => {
        if (config.method?.toLowerCase() === 'get') {
          const cacheKey = this.generateCacheKey(config);
          if (this.cache.has(cacheKey)) {
            return Promise.reject({ cached: true, data: this.cache.get(cacheKey) });
          }

          const requestPromise = this.axiosInstance(config);
          this.cache.set(cacheKey, requestPromise);
          return config;
        }
        return config;
      });

      // 에러 처리 인터셉터
      this.axiosInstance.interceptors.response.use(
        (response) => response.data,
        (error: AxiosError) => {
          if (error.response) {
            // 서버 오류 (4xx, 5xx)
            throw {
              status: error.response.status,
              message: error.response.data?.message || 'Server Error',
            };
          } else if (error.request) {
            // 네트워크 오류
            throw { status: 0, message: 'Network Error' };
          } else if ((error as any).cached) {
            // 캐시 히트
            return (error as any).data;
          } else {
            // 기타 오류
            throw { status: -1, message: error.message };
          }
        }

        // ps. 보편적으로 401인증 에러 발생 시에는 jwt 리프레쉬 요청 로직이 해당 부분에 구성이 되었습니다.
      );
    }

    public async get<T>(url: string, params?: object): Promise<T> {
      return this.axiosInstance.get(url, { params });
    }

    public async post<T>(url: string, data?: object): Promise<T> {
      return this.axiosInstance.post(url, data);
    }
  }

  // 싱글톤 인스턴스 생성
  // 보통 퍼사드는 특징상 많은 기능을 갖고 있는 구조이다 보니 
  // 전역에서 일관되게 사용되는 경우가 잦습니다. === 싱글톤
  export const httpClient = new HttpClient(process.env.API_BASE_URL || '');
  ```
  - 실제 프로젝트에서 해당 코드를 조금 더 확장한다면?
    - JWT 토큰 자동 주입
    - 로깅 시스템 연동
    - 모의 API (Mocking) 지원