---
title: "(CI/CD) Github Actions Secret의 JSON 파일 저장 오류"
tags:
    - java
date: "2024-12-14"
thumbnail: "/assets/img/thumbnail/sample.png"
---
> *구글 인앱결제 서비스 로직을 구현할 때 겪은 오류이다.*

## 1. 오류 발생 상황
### 전체 코드
```java
// InAppPurchaseService.java
@Service
public class InAppPurchaseService {

  @Value("${google-account.file-path}")
  private String googleAccountFilePath;

  @Value("${google-application.package-name}")
  private String googleApplicationPackageName;

  @Transactional
  public Boolean validateReceipt(Long userId, PurchaseRequest purchaseRequest) {
    HttpTransport httpTransport;
    GoogleCredentials credentials;
    try {
      httpTransport = GoogleNetHttpTransport.newTrustedTransport();
      InputStream inputStream = new ClassPathResource(googleAccountFilePath).getInputStream();
      credentials = GoogleCredentials
	  		.fromStream(inputStream)
			.createScoped(AndroidPublisherScopes.ANDROIDPUBLISHER);
    } catch (IOException | GeneralSecurityException e) {
      throw ExpectedException.withLogging(ResponseCode.GoogleCredentialException, e);	
    }
	
    AndroidPublisher.Builder builder = new AndroidPublisher.Builder(httpTransport, 
					GsonFactory.getDefaultInstance(), 
					new HttpCredentialsAdapter(credentials));
    AndroidPublisher publisher = builder.setApplicationName(googleApplicationPackageName).build();
    //...(중략)
  }
}
```

```yaml
# ci-cd.yml
jobs:
  CI-CD:
    runs-on: ubuntu-22.04
    steps:
	# ...(중략)
      - name: Create Google key.json file
	if: contains(github.ref, 'main') || contains(github.ref, 'staging')
	run: |
	  cd ./genti-api/src/main/resources
  	  mkdir -p ./jsonkey
	  echo "${secrets.GOOGLE_ACCOUNT_KEY }" > ./jsonkey/key.json
	shell: bash
	# ...(중략)
```

### 오류 메세지
- com.google.gson.stream.<span style="color:red">MalformedJsonException</span>: Use JsonReader.setLenient(true) to accept malformed JSON at line 2 column 4 path $.

## 2. 오류 분석
### 코드 오류 부분
```java
// InAppPurchaseService.java
public Boolean validateReceipt(Long userId, PurchaseRequest purchaseRequest) {
  //...
  try {
    //...
    credentials = GoogleCredentials
		.fromStream(inputStream)  // 오류 발생 부분
		.createScoped(AndroidPublisherScopes.ANDROIDPUBLISHER);
  }
  //...
}
```

### 오류 분석
1. 오류 메시지를 보면, JSON 파싱 중 문제가 있다고 한다. **로컬에서는 발생하지 않던 오류라서, 서버 측에 저장되는 JSON 파일에 문제가 있을 것으로 추측**했다.

2. 따라서, 서버 측에 저장되는 JSON 파일을 확인하기 위해 해당 JSON 파일을 S3에 저장하는 부분을 ci-cd.yml에 임시로 추가했다.  
```yml
- name: Upload key.json to S3
	 if: contains(github.ref, 'staging')
	 run: |
	  aws s3 cp ./genti-api/src/main/resources/jsonkey/key.json s3://$S3_BUCKET_NAME/jsonkey/key.json
```

3. S3에서 key.json 파일을 다운받아 확인해보았더니, 아래처럼 큰따옴표가 다 사라져 있었다.	
	```
	{
	  type: service_account,
	  project_id: example,
	  (...)
	}
	```

4. **<span style="color:red">결론 - 오류의 원인</span>**
	- Github Actions Secret에 JSON 파일 내용을 넣으면 JSON 파일 내용의 큰따옴표가 다 사라지게 된다. 
		
## 3. 오류 해결
### [create-json](https://github.com/marketplace/actions/create-json) 라이브러리 사용 
```yml
# ci-cd.yml
- name: Create jsonkey directory
  if: contains(github.ref, 'main') || contains(github.ref, 'staging')
  run: mkdir -p ./genti-api/src/main/resources/jsonkey

- name: Create Google key.json file
  if: contains(github.ref, 'main') || contains(github.ref, 'staging')  
  id: create-json
  uses: jsdaniell/create-json@1.1.2
  with:
    name: "./genti-api/src/main/resources/jsonkey/key.json"
    json: ${ secrets.GOOGLE_ACCOUNT_KEY }
```

---
- 참조 자료
https://stackoverflow.com/questions/11484353/gson-throws-malformedjsonexception  
https://choo.oopy.io/fd2d4fc6-21ac-45b6-bd0c-05768920bb00  
https://roel-yomojomo.tistory.com/38  
https://velog.io/@godkimchichi/Github-Actions-secret%EC%97%90-json-%EB%84%A3%EA%B3%A0-%EC%8B%B6%EC%9D%84-%EB%95%8C  