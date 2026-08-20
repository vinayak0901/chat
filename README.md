rsaCipher ALGO: RSA/ECB/OAEPWithSHA-256AndMGF1Padding
rsaCipher PROVIDER: SunJCE version 21
encryptedAESKey LEN: 344
serverPrivateKey ALGO: RSA
serverPrivateKey CLASS: class sun.security.rsa.RSAPrivateCrtKeyImpl
serverPrivateKey LEN: 2048
encryptedAESKey bytes LEN: 256
rsaCipher PARAMETERS: MD: SHA-256


	public static byte[] encryptAESKey(SecretKey aesKey, String serverCertificatePath) {
		try (FileInputStream fisServer = new FileInputStream(serverCertificatePath)) {
			CertificateFactory cf = CertificateFactory.getInstance("X.509");
			X509Certificate certificate = (X509Certificate) cf.generateCertificate(fisServer);
			PublicKey serverPublicKey = certificate.getPublicKey();

			Cipher rsaCipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
			rsaCipher.init(Cipher.ENCRYPT_MODE, serverPublicKey);

			return rsaCipher.doFinal(aesKey.getEncoded());
		} catch (Exception e) {
			System.out.println("Error in encryptAESKey.");
			e.printStackTrace();
		}
		return null;
	}


OAEPParameterSpec oaepParams = new OAEPParameterSpec(
    "SHA-256",
    "MGF1",
    MGF1ParameterSpec.SHA256,
    PSource.PSpecified.DEFAULT
);

Cipher rsaCipher = Cipher.getInstance("RSA/ECB/OAEPPadding");

rsaCipher.init(
    Cipher.DECRYPT_MODE,
    serverPrivateKey,
    oaepParams
);

byte[] encrypted = publicKey.Encrypt(
    aesKeyBytes,
    RSAEncryptionPadding.OaepSHA256
);

Asymmetric Encryption Specification
Parameter
Value
Algorithm
RSA
Encryption Scheme
RSA-OAEP
OAEP Digest
SHA-256
Mask Generation Function
MGF1
MGF1 Digest
SHA-256
OAEP Label
Empty / Default
RSA Key
RSA public key from X.509 certificate
Purpose
Encryption of AES symmetric key
Complete specification:
RSA-OAEP encryption with SHA-256 as the OAEP digest, MGF1 using SHA-256 as the MGF1 digest, and an empty OAEP label.
