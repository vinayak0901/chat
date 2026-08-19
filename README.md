rsaCipher ALGO: RSA/ECB/OAEPWithSHA-256AndMGF1Padding
rsaCipher PROVIDER: SunJCE version 21
encryptedAESKey LEN: 344
serverPrivateKey ALGO: RSA
serverPrivateKey CLASS: class sun.security.rsa.RSAPrivateCrtKeyImpl
serverPrivateKey LEN: 2048
encryptedAESKey bytes LEN: 256
rsaCipher PARAMETERS: MD: SHA-256

java.lang.NullPointerException: Cannot invoke "String.getBytes(java.nio.charset.Charset)" because "src" is null
        at java.base/java.util.Base64$Decoder.decode(Base64.java:593) ~[na:na]
        at in.co.sbi.rateengine.fixccilfeed.service.CryptoService.verifySignature(CryptoService.java:147) ~[classes/:na]
        at in.co.sbi.rateengine.fixccilfeed.service.CryptoService.processPasswordDecryption(CryptoService.java:67) ~[classes/:na]
        at in.co.sbi.rateengine.fixccilfeed.engine.FIXEngine.fromAdmin(FIXEngine.java:81) ~[classes/:na]
        at quickfix.Session.fromCallback(Session.java:1867) ~[quickfixj-all-2.3.2.jar:2.3.2]
        at quickfix.Session.verify(Session.java:1813) ~[quickfixj-all-2.3.2.jar:2.3.2]
        at quickfix.Session.nextLogon(Session.java:2149) ~[quickfixj-all-2.3.2.jar:2.3.2]
        at quickfix.Session.next(Session.java:1050) ~[quickfixj-all-2.3.2.jar:2.3.2]
        at quickfix.Session.next(Session.java:1228) ~[quickfixj-all-2.3.2.jar:2.3.2]
        at quickfix.mina.ThreadPerSessionEventHandlingStrategy$MessageDispatchingThread.doRun(ThreadPerSessionEventHandlingStrategy.java:222) ~[quickfixj-all-2.3.2.jar:2.3.2]
        at quickfix.mina.ThreadPerSessionEventHandlingStrategy$ThreadAdapter.run(ThreadPerSessionEventHandlingStrategy.java:146) ~[quickfixj-all-2.3.2.jar:2.3.2]
        at java.base/java.lang.Thread.run(Thread.java:1583) ~[na:na]
