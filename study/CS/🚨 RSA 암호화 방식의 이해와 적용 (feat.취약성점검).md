# 🚨 RSA 암호화 방식의 이해와 적용 (feat.취약성점검)

#공부 #Tokken #Security #SPRING #보안

---

## RSA 암호화는 **비대칭 키 암호화 방식** 중 하나이다.

1977년에 **Rivest, Shamir, Adleman** 이라는 세 명의 수학자가 고안해서 RSA라고 부른다.
이 방식은 지금도 SSL/TLS, 이메일, 디지털 서명, 인증서 등 **보안의 핵심 인프라**에서 계속 쓰이고 있는 방식이다.

# 정의

- RSA는 "공개키로 암호화하고, 개인키로 복호화" 또는 그 반대로 동작하는 비대칭 암호화 알고리즘이다.

---

# 핵심 개념

| 항목        | 설명                                  |
| --------- | ----------------------------------- |
| **키쌍**    | 공개키 (Public Key), 개인키 (Private Key) |
| **암호화**   | 송신자가 수신자의 **공개키**로 암호화              |
| **복호화**   | 수신자는 자신의 **개인키**로 복호화               |
| **보안 근거** | 큰 수의 소인수분해가 **매우 어렵다는 수학적 문제**에 기반  |
| **용도**    | 데이터 암호화, 디지털 서명, 인증 등               |

---

# 그래서 왜 보안에 효과적인걸까?

- 공개키로 암호화된 데이터는 **개인키를 소유한 공개키 생성자만이 복호화** 가능하다.
- **사용자( Client )마다 공개키와 개인키를 접근 시 새로 생성/발급하여 원본 데이터 추적이 불가**하다.

>[!tip] 결론
> 즉, 클라이언트의 요청이 들어오면 서버에서 *공개키(클라이언트에게 제공할 키)* 와 *개인키(서버측에서 갖고있는, 공개키와 한쌍이 되는 키)* 를 매번 새로 발급하기 때문에, 추적이 어렵고 암호화에 뛰어나다.

#### 단, AES(대칭 암호화)와 방식보다 성능(속도)측면에서 떨어진다.
- 서비스 특성에 따라 적절하게 적용할 필요가 있다.

---

# Java/JS 환경에서 RSA암호화 적용하기

>[!tip] 정보
> - **RSA 키 객체 (`PublicKey`, `PrivateKey`)** 는 메모리 상에서만 동작할 수 있다.
> - 우리가 파일, 네트워크, DB로 키를 주고받을 때는 *Base64* 문자열 로 직렬화해야 해야한다.

#### 전반적인 로직 흐름
[로그인 / 회원가입 페이지]
1. `클라이언트`에서 `서버`에 공개키 요청
2. `서버`에서 키쌍 과 KeyUUID 생성
3. 개인키는 KeyUUID와 함께 캐쉬에 저장 / 공개키는 KeyUUID와 `클라이언트`에 전달
4. `클라이언트`는 `서버`로부터 받은 공개키로 평문 암호화, `서버`에 전달
5. 암호화된 평문을 KeyUUID로 찾은 공개키로 복호화 -> 평문 완성

# 1. 키 쌍 발급 로직

### 하나의 쌍으로 이루어지는 *공개키(public key)* 와 *개인키(private key)* 를 생성하는 로직 

#### **`java.security`** 패키지를 사용하여 구현한다. **Java의 보안 프레임워크의 핵심**
- 암호화, 해시, 키 생성, 인증서 처리, 서명 등 **암호학 기반 기능들을 제공**하는 클래스들의 모음이다.

```java
    private static final String INSTANCE_TYPE = "RSA";  
  
    // 2048bit RSA KeyPair 생성.  
    public static KeyPair generateKeypair() throws NoSuchAlgorithmException {  
  
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance(INSTANCE_TYPE);  
        keyPairGen.initialize(2048, new SecureRandom());  
  
        return keyPairGen.genKeyPair();  
    }  
```
- KeyPairGenerator (공개키/개인키 쌍 생성) 을 사용한다.
* `NoSuchAlgorithmException` : 지정한 알고리즘 이름이 현재 JVM 환경에서 지원되지 않거나 잘못된 경우 발생하는 **체크 예외**다.
- `KeyPair`타입 : PrivateKey와 PublicKey로 이루어져있는 데이터 타입
- `SecureRandom`을 시드로 사용해 보안 수준 향상

 2048bit로 RSA암호화 방식을 사용하여 **keyPair**를 생성하는 코드이다.

---

# 2.1 평문 + base64공개키 -> base64암호문 생성 로직 .Java

```java
	private static final String INSTANCE_TYPE = "RSA";
	
	// 평문 + 공개키 Base64로 암호문 생성  
	public static String rsaEncode(String plainText, String publicKey)  
	        throws InvalidKeyException, InvalidKeySpecException, NoSuchAlgorithmException, NoSuchPaddingException, IllegalBlockSizeException, BadPaddingException {  
	  
	    Cipher cipher = Cipher.getInstance(INSTANCE_TYPE);  
	    cipher.init(Cipher.ENCRYPT_MODE, convertPublicKey(publicKey));  
	  
	    byte[] plainTextByte = cipher.doFinal(plainText.getBytes());  
	  
	    return base64EncodeToString(plainTextByte);  
	}  
	//Base64 공개키 -> 공개키로 디코딩  
	public static PublicKey convertPublicKey(String publicKey)  
	        throws InvalidKeySpecException, NoSuchAlgorithmException {  
	  
	    KeyFactory keyFactory = KeyFactory.getInstance(INSTANCE_TYPE);  
	    byte[] publicKeyByte = Base64.getDecoder().decode(publicKey.getBytes());  
	  
	    return keyFactory.generatePublic(new X509EncodedKeySpec(publicKeyByte));  
	}
```

- `Cipher` : Java 보안 API에서 실제 *암호화/복호화*를 수행하는 핵심 클래스
	AES, RAS, DES 같은 알고리즘을 직접 실행하는 암호 모듈 , 암호화 엔진 이다.

- `Cipher cipher = Cipher.getInstance(INSTANCE_TYPE);` : 타입에 따라, 암호화 모드와 패딩 방식이 결정된다. 
- `convertPublicKey` : base64기반 코드를 실제 키 객체로 변환
- `KeyFactory` : 키 복원용 펙토리 객체
- `keyFactory.generate...` : 실제 키 객체 생성
- `X509EncodedKeySpec` → 공개키 표준 포맷 스펙

---
# 2.2 평문 + base64공개키 -> base64암호문 생성 로직 .JS

>[!tip] node-forge 패키지를 사용한다.
>  base64, encode등 TLS프로토콜(암호화 도구)를 구현한 패키지 이다.

```JavaScript
<script src="https://cdn.jsdelivr.net/npm/node-forge@1.3.1/dist/forge.min.js"></script>

<script>
/**
 * 서버에서 공개키를 받아서 RSA로 암호화하는 함수
 * @param {string} plainText - 암호화할 평문
 * @param {string} publicKeyBase64 - 서버로부터 받은 Base64 인코딩된 공개키
 * @returns {string} 암호화된 Base64 문자열
 */
function rsaEncryptWithBase64PublicKey(plainText, publicKeyBase64) {
  const forge = window.forge;

  // 1. Base64 디코딩 → DER 바이너리
  const der = forge.util.decode64(publicKeyBase64);

  // 2. DER → ASN.1 파싱 → PublicKey 객체
  const asn1 = forge.asn1.fromDer(der);
  const publicKey = forge.pki.publicKeyFromAsn1(asn1);

  // 3. RSA 암호화
  const encryptedBytes = publicKey.encrypt(plainText, 'RSAES-PKCS1-V1_5');

  // 4. 암호문을 Base64 인코딩해서 반환
  return forge.util.encode64(encryptedBytes);
}
</script>
```
- 클라이언트 단에서 request전달 전, 암호화 하기 위한 코드 / 동작 구성은 `2.1`과 동일 하다

---

# 3. base64암호문 + base64개인키 -> 평문 생성 로직
```java
	private static final String INSTANCE_TYPE = "RSA";

	// 암호문 + 개인키 Base64로 평문 생성  
	public static String rsaDecode(String encryptedPlainText, String privateKey)  
	        throws NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeyException, InvalidKeySpecException, IllegalBlockSizeException, BadPaddingException {  
	  
	    byte[] encryptedPlainTextByte = Base64.getDecoder().decode(encryptedPlainText.getBytes());  
	  
	    Cipher cipher = Cipher.getInstance(INSTANCE_TYPE);  
	    cipher.init(Cipher.DECRYPT_MODE, convertPrivateKey(privateKey));  
	  
	    return new String(cipher.doFinal(encryptedPlainTextByte));  
	}  
	//Base64 개인키 -> 개인키로 디코딩  
	public static PrivateKey convertPrivateKey(String privateKey)  
	        throws InvalidKeySpecException, NoSuchAlgorithmException {  
	  
	    KeyFactory keyFactory = KeyFactory.getInstance(INSTANCE_TYPE);  
	    byte[] privateKeyByte = Base64.getDecoder().decode(privateKey.getBytes());  
	  
	    return keyFactory.generatePrivate(new PKCS8EncodedKeySpec(privateKeyByte));  
	}
```

- `Cipher` 객체를 사용하여 위와 동일하게 동작.
- `PKCS8EncodedKeySpec` → 개인키 표준 포맷 스펙

---

# + 바이너리 데이터를 Base64로 인코딩 .java

```java
  public static String base64EncodeToString(byte[] byteData) {
    return Base64.getEncoder().encodeToString(byteData);
  }
```
- cipher.doFinal(...) 과 같은 코드는 *바이너리 데이터*로 리턴값을 보낸다.

---

# 백로직 및 프론트 검증 완료

### *junit*과 *assertj*를 사용하여 검증
- `JUnit` → 테스트 프레임워크  
- `AssertJ` → 테스트 결과를 검증(assert)할 때 쓰는 **강력한 assertion 도구**

JS코드는 제외
```java
private static final String PLAIN_TEXT = "키 암/복호화 테스트 123 abc !@#";

@Test  
@DisplayName("RSA 키쌍 생성 및 암/복호화 통합 테스트")  
public void testGenerateKeypairAndEncryptDecrypt() throws Exception {  
    // 키쌍 생성  
    KeyPair keyPair = rsaService.generateKeypair();  
    PublicKey publicKey = keyPair.getPublic();  
    PrivateKey privateKey = keyPair.getPrivate();  
  
    // 공개키, 개인키 → Base64 인코딩  
    String publicKeyBase64 = rsaService.base64EncodeToString(publicKey.getEncoded());  
    String privateKeyBase64 = rsaService.base64EncodeToString(privateKey.getEncoded());  
    System.out.println("공개키Base64 : " + publicKeyBase64);  
    System.out.println("개인키Base64 : " + privateKeyBase64);  
  
    // 암호화  
    String encryptedText = rsaService.rsaEncode(PLAIN_TEXT, publicKeyBase64);  
    System.out.println("RSA암호화 텍스트 : " + encryptedText);  
  
    // 복호화 (개인키 사용해야 함)  
    String decryptedText = rsaService.rsaDecode(encryptedText, privateKeyBase64);  
    System.out.println("RSA복호화 텍스트 : " + decryptedText);  
  
    // 검증  
    Assertions.assertThat(decryptedText).isEqualTo(PLAIN_TEXT);  
}
```
결과
![[Pasted image 20250325102427.png]]
- Base64기반 String 변환 및 암/복호화 테스트 완료
- <u>js암호화도 동일하게 동작 확인</u>

---

## *+ 로그인 / 회원가입 서비스 적용 완료*
![[Pasted image 20250325165845.png]]
![[Pasted image 20250325172818.png]]
