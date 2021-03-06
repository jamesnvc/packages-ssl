dnl Process this file with autoconf to produce a configure script.

AC_INIT(install-sh)
AC_PREREQ([2.50])
AC_CONFIG_HEADER(config.h)

AC_SUBST(TARGETS)
AC_SUBST(PLTARGETS)

m4_include([../ac_swi_c.m4])

dnl MacOS hacks.  There is both libssl.dylib in /usr/lib and in the
dnl Macport or Homebrew.  MacOS discontinued support for OpenSSL, so
dnl we have to find the right version.

case "$PLARCH" in
    *-win32|*-win64)
        AC_CHECK_LIB(ws2_32, main)
        AC_CHECK_LIB(gdi32, main)
	AC_CHECK_LIB(crypt32, main)
        ;;
    *darwin*)
	LIBS="$LIBS -framework CoreFoundation -framework Security"
	if test -z "$SSLPREFIX"; then
          if test -f /opt/local/lib/libssl.$SO; then
            SSLPREFIX=/opt/local
          else
            SSLPREFIX=/usr/local
          fi
        fi
	;;
esac

AC_CHECK_HEADERS(unistd.h sys/types.h sys/time.h sys/select.h fcntl.h)
AC_CHECK_HEADERS(Security/Security.h)

AC_MSG_CHECKING(kSecClass)
AC_TRY_COMPILE(
[ #include <sys/types.h>
  #include <Security/Security.h>
],
[ const void *key = kSecClass;
], AC_DEFINE(HAVE_KSECCLASS, 1,
	     [Define if Security/Security.h supplies kSecClass])
   AC_MSG_RESULT(yes),
   AC_MSG_RESULT(no))

if test ! -z "$SSLPREFIX"; then
  AC_MSG_CHECKING(Whether setting for SSLPREFIX is ok)
  if test -f "$SSLPREFIX/include/openssl/ssl.h" -a \
          -f "$SSLPREFIX/lib/libssl.$SO"; then
    AC_MSG_RESULT(Using $SSLPREFIX)
    LIBS="$LIBS $SSLPREFIX/lib/libssl.$SO $SSLPREFIX/lib/libcrypto.$SO"
    CFLAGS="-I/$SSLPREFIX/include $CFLAGS"
    AC_CHECK_FUNCS(SSL_library_init OPENSSL_init_ssl)
    SSL_ok=yes
  else
    AC_MSG_RESULT(no)
    SSLPREFIX=
  fi
fi

if test -z "$SSLPREFIX"; then
  AC_CHECK_LIB(crypto, main)
  AC_CHECK_LIB(ssl, SSL_library_init)
  AC_CHECK_LIB(ssl, OPENSSL_init_ssl)

  AC_CHECK_HEADERS(openssl/ssl.h)

  if ( test "$ac_cv_lib_ssl_SSL_library_init" = yes ||
       test "$ac_cv_lib_ssl_OPENSSL_init_ssl" = yes ) &&
     test "$ac_cv_lib_crypto_main" = yes; then
    SSL_ok=yes
  fi
fi

if test "$SSL_ok" = yes; then
  TARGETS="ssl4pl.$SO crypto4pl.$SO"
  PLTARGETS="crypto.pl ssl.pl saml.pl xmldsig.pl xmlenc.pl"
else
  TARGETS=""
  PLTARGETS=""
fi

AC_CHECK_TYPES(socklen_t, [], [],
[
#include <sys/types.h>
#include <sys/socket.h>
])

AC_MSG_CHECKING(h_errno)
AC_TRY_COMPILE(
[
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include <netinet/in.h>
],
[ int x = h_errno;
], AC_DEFINE(HAVE_H_ERRNO, 1,
	     [Define of h_errno is provided])
   AC_MSG_RESULT(yes),
   AC_MSG_RESULT(no))

AC_CHECK_FUNCS(timegm ERR_remove_state ERR_remove_thread_state)
AC_CHECK_FUNCS(X509_check_host)
AC_CHECK_FUNCS(CRYPTO_THREADID_get_callback CRYPTO_THREADID_set_callback)
AC_CHECK_FUNCS(EVP_MD_CTX_free OPENSSL_zalloc)
AC_CHECK_FUNCS(X509_CRL_get0_signature X509_get0_signature)
AC_CHECK_FUNCS(X509_get0_notBefore X509_get0_notAfter)
AC_CHECK_FUNCS(X509_digest X509_CRL_digest)
AC_CHECK_FUNCS(X509_STORE_CTX_get0_chain)
AC_CHECK_FUNCS(i2d_re_X509_tbs)
AC_CHECK_FUNCS(OpenSSL_version)
AC_CHECK_FUNCS(EVP_CIPHER_CTX_reset)
AC_CHECK_FUNCS(EVP_blake2b512 EVP_blake2s256)
AC_CHECK_FUNCS(EVP_sha3_224 EVP_sha3_256 EVP_sha3_384 EVP_sha3_512)
AC_CHECK_FUNCS(HMAC_CTX_new HMAC_CTX_free)

AC_CHECK_HEADERS(openssl/kdf.h)

if test "x$ac_cv_func_X509_get0_signature" = "xyes"; then
  ac_oldcflags="$CFLAGS"
  CFLAGS="$CFLAGS -Werror"
  AC_MSG_CHECKING("whether X509_get0_signature takes const")
  AC_TRY_COMPILE(
[
#include <openssl/x509.h>
],
		 [ ASN1_BIT_STRING *psig;
		   X509_ALGOR *palg;
		   const X509 *data;
		   X509_get0_signature(&psig, &palg, data);
		 ],
		 [AC_MSG_RESULT(no);  GET0SIG_CONST_T=""],
		 [AC_MSG_RESULT(yes); GET0SIG_CONST_T="const"])
CFLAGS=$ac_oldcflags
else
	GET0SIG_CONST_T=const
fi
AC_DEFINE_UNQUOTED(GET0SIG_CONST_T, $GET0SIG_CONST_T,
		   [Define X509_get0_signature const args])

AC_CHECK_TYPES(CRYPTO_THREADID, [], [],
[
#include <openssl/ssl.h>
])

AC_CHECK_MEMBERS(X509_VERIFY_PARAM.id,
[ AC_DEFINE(HAVE_X509_VERIFY_PARAM_ID, 1,
	    [Define of X509_VERIFY_PARAM_ID is provided])
], [],
[
#include <openssl/ssl.h>
])

AC_MSG_CHECKING(system certificate file)
SYSTEM_CACERT_FILENAME=/etc/ssl/certs/ca-certificates.crt
cert_candidates="/etc/ssl/certs/ca-certificates.crt"
cert_candidates="$cert_candidates /etc/pki/tls/certs/ca-bundle.crt"
cert_candidates="$cert_candidates /etc/ssl/ca-bundle.pem"
cert_candidates="$cert_candidates /etc/ssl/cert.pem"

for f in $cert_candidates; do
  if test -f $f; then
    SYSTEM_CACERT_FILENAME=$f
    break
  fi
done
AC_DEFINE_UNQUOTED(SYSTEM_CACERT_FILENAME, "$SYSTEM_CACERT_FILENAME",
		   [Location of the system certificate file])
if test -f $SYSTEM_CACERT_FILENAME; then
  AC_MSG_RESULT($SYSTEM_CACERT_FILENAME)
else
  AC_MSG_RESULT($SYSTEM_CACERT_FILENAME [(does not exist)])
fi

AC_OUTPUT(Makefile)




