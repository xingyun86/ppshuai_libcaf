http://actor-framework.org/

https://github.com/actor-framework/actor-framework.git

https://github.com/xingyun86/actor-framework.git

Patch:

1. actor-framework\libcaf_openssl\src\manager.cpp:64,

static CRYPTO_dynlock_value* dynlock_create(const char* /* file */,
                                            int /* line */) {
-//return new CRYPTO_dynlock_value{};
+return new CRYPTO_dynlock_value();
}

2. actor-framework\libcaf_openssl\src\middleman_actor.cpp:46,

#ifdef CAF_WINDOWS
-//# include <winsock2.h>
-//# include <ws2tcpip.h> // socket_size_type, etc. (MSVC20xx)

+//#if(_WIN32_WINNT >= 0x0600)
+
+//
+// Address families.
+//
+
+typedef USHORT ADDRESS_FAMILY;
+
+//#endif//(_WIN32_WINNT >= 0x0600)
+
+//
+// Portable socket structure (RFC 2553).
+//
+
+//
+// Desired design of maximum size and alignment.
+// These are implementation specific.
+//
+#define _SS_MAXSIZE 128                 // Maximum size
+#define _SS_ALIGNSIZE (sizeof(__int64)) // Desired alignment

+//
+// Definitions used for sockaddr_storage structure paddings design.
+//

+#if(_WIN32_WINNT >= 0x0600)
+#define _SS_PAD1SIZE (_SS_ALIGNSIZE - sizeof(USHORT))
+#define _SS_PAD2SIZE (_SS_MAXSIZE - (sizeof(USHORT) + _SS_PAD1SIZE + _SS_ALIGNSIZE))
+#else
+#define _SS_PAD1SIZE (_SS_ALIGNSIZE - sizeof (short))
+#define _SS_PAD2SIZE (_SS_MAXSIZE - (sizeof (short) + _SS_PAD1SIZE \
+                                                    + _SS_ALIGNSIZE))
+#endif //(_WIN32_WINNT >= 0x0600)
+
+typedef struct sockaddr_storage {
+	ADDRESS_FAMILY ss_family;      // address family
+
+	CHAR __ss_pad1[_SS_PAD1SIZE];  // 6 byte pad, this is to make
+								   //   implementation specific pad up to
+								   //   alignment field that follows explicit
+								   //   in the data structure
+	__int64 __ss_align;            // Field to force desired structure
+	CHAR __ss_pad2[_SS_PAD2SIZE];  // 112 byte pad to achieve desired size;
+								   //   _SS_MAXSIZE value minus size of
+								   //   ss_family, __ss_pad1, and
+								   //   __ss_align fields is 112
+} SOCKADDR_STORAGE_LH, *PSOCKADDR_STORAGE_LH, FAR *LPSOCKADDR_STORAGE_LH;
+
#else
# include <sys/types.h>
# include <sys/socket.h>
#endif