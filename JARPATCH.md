
Client patching
===============

To use this extension by patching the client first download `frmall.jar` and unzip it's contents. Then use your favorite Java decompiler to decompile the following classes:
* `oracle.forms.net.EncryptedInputStream`
* `oracle.forms.net.EncryptedOutputStream`

Remove all lines containing an XOR operation, like this: 

```
--- frmall_src_original/oracle/forms/net/EncryptedInputStream.java  2017-08-19 11:16:57.000000000 +0200
+++ frmall_src/oracle/forms/net/EncryptedInputStream.java   2017-08-19 11:29:56.083779579 +0200
@@ -163,7 +163,7 @@
         arrayOfInt[m] = i1; 
         int tmp184_182 = n;
         byte[] tmp184_179 = this.mBuf;
-        tmp184_179[tmp184_182] = ((byte)(tmp184_179[tmp184_182] ^ arrayOfInt[((arrayOfInt[k] + i1) % 256)]));
+        //tmp184_179[tmp184_182] = ((byte)(tmp184_179[tmp184_182] ^ arrayOfInt[((arrayOfInt[k] + i1) % 256)]));
       }   
       this.mI = k;
       this.mJ = m;
```

Then recompile the classes after moving them in their package in the unzipped (compiled) package contents (you'll probably also need to fix some minor errors generated by the decompiler):

```
javac EncryptedInputStream.java
javac EncryptedOutputStream.java
```

Update `frmall.jar` with the updated `.class` files, and delete `META-INF/ORA*` so code signing won't be a problem for JRE. Place the resulting `.jar` file to `/tmp/frmall.jar` (this is a hardcoded path in the MiMproxy script, you can edit the source if you don't like it). Also copy frmall.jar to the `lib/` directory of the OracleFormsSerializer. Then you can compile the Burp extension with the provided Ant script.

To enable this legacy mode start the script with the `corrupt_handshake` option set to False:

```
mitmdump -s mitmproxy_oracleforms.py --set corrupt_handshake=false -p 8081
```