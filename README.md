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
