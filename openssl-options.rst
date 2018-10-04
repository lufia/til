OpenSSLのオプション
===================

.. highlight:: console

秘密鍵に設定しているパスフレーズを外す場合、
インターネットで調べると以下のコマンドを見かけるが::

	$ openssl rsa -in key.pem -out nopass-key.pem

標準出力に出力したいなら、
単純に ``-out`` オプションを付けなければいい::

	$ openssl rsa -in key.pem

PEMの文字列
-----------

PEMの **---- BEGIN XX ----** から **---- END XX ----** で使われる文字。

.. code-block:: c

	#define PEM_STRING_X509_OLD     "X509 CERTIFICATE"
	#define PEM_STRING_X509         "CERTIFICATE"
	#define PEM_STRING_X509_PAIR    "CERTIFICATE PAIR"
	#define PEM_STRING_X509_TRUSTED "TRUSTED CERTIFICATE"
	#define PEM_STRING_X509_REQ_OLD "NEW CERTIFICATE REQUEST"
	#define PEM_STRING_X509_REQ     "CERTIFICATE REQUEST"
	#define PEM_STRING_X509_CRL     "X509 CRL"
	#define PEM_STRING_EVP_PKEY     "ANY PRIVATE KEY"
	#define PEM_STRING_PUBLIC       "PUBLIC KEY"
	#define PEM_STRING_RSA          "RSA PRIVATE KEY"
	#define PEM_STRING_RSA_PUBLIC   "RSA PUBLIC KEY"
	#define PEM_STRING_DSA          "DSA PRIVATE KEY"
	#define PEM_STRING_DSA_PUBLIC   "DSA PUBLIC KEY"
	#define PEM_STRING_PKCS7        "PKCS7"
	#define PEM_STRING_PKCS7_SIGNED "PKCS #7 SIGNED DATA"
	#define PEM_STRING_PKCS8        "ENCRYPTED PRIVATE KEY"
	#define PEM_STRING_PKCS8INF     "PRIVATE KEY"
	#define PEM_STRING_DHPARAMS     "DH PARAMETERS"
	#define PEM_STRING_DHXPARAMS    "X9.42 DH PARAMETERS"
	#define PEM_STRING_SSL_SESSION  "SSL SESSION PARAMETERS"
	#define PEM_STRING_DSAPARAMS    "DSA PARAMETERS"
	#define PEM_STRING_ECDSA_PUBLIC "ECDSA PUBLIC KEY"
	#define PEM_STRING_ECPARAMETERS "EC PARAMETERS"
	#define PEM_STRING_ECPRIVATEKEY "EC PRIVATE KEY"
	#define PEM_STRING_PARAMETERS   "PARAMETERS"
	#define PEM_STRING_CMS          "CMS"
