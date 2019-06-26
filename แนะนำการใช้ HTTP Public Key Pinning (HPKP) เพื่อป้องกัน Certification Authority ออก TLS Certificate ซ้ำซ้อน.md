# แนะนำการใช้ HTTP Public Key Pinning (HPKP) เพื่อป้องกัน Certification Authority ออก TLS Certificate ซ้ำซ้อน

กระบวนการรับรอง SSL/TLS Certificate หรือ “ใบรับรองอิเล็กทรอนิกส์” (ต่อไปจะเรียกว่า TLS Certificate) จะผ่านทาง Certificate Authority หรือ “ผู้ออกใบรับรองอิเล็กทรอนิกส์” (CA) โดยเว็บเบราว์เซอร์หรือระบบปฏิบัติการ (ต่อไปจะเรียกว่า client) จะมีรายการที่เรียกว่า Trusted Root Certification Authorities หรือ “รายการใบรับรองอิเล็กทรอนิกส์ของผู้ออกใบรับรองอิเล็กทรอนิกส์สำหรับผู้อื่น” อยู่ภายใน โดยมากมักอ้างอิงกับตัวระบบปฏิบัติการเป็นหลัก ซึ่งจะบรรจุ Root Certificate หรือ “ใบรับรองอิเล็กทรอนิกส์ลำดับชั้นบนสุด” เพื่อให้ client สามารถเชื่อใบรับรองอิเล็กทรอนิกส์ที่ได้รับรองจาก Certificate Authority เหล่านั้น (Trust any certificates issued by the Certificate Authorities)

การที่ Certificate Authority จะสามารถอยู่ในรายการดังกล่าวได้ จะต้องผ่านการตรวจสอบคุณสมบัติตามมาตรฐานจากหน่วยงาน [CA/Browser Forum](https://cabforum.org/) เพราะผู้ผลิต client ต่างๆ จะใช้ข้อมูลจาก CA/Browser Forum ในการบรรจุ Root Certificate ลงไปในซอฟต์แวร์ของตน ([สมาชิกของ CA/Browser Forum](https://cabforum.org/members/))

[![](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/640px-Chain_of_trust.svg_-e1465485393532.png)](https://www.thaicyberpoint.com/ford/blog/id/5008/640px-chain_of_trust-svg/#main)

รูปภาพจาก [Root certificate](https://en.wikipedia.org/wiki/Root_certificate)

[![](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/2016-06-09_212212-e1465485450292.png)](https://www.thaicyberpoint.com/ford/blog/?attachment_id=5009#main)

รายการ Trusted Root Certification Authorities ภายใน operating system ของ Windows

**ปัญหาที่ระดับ Trusted Root Certification Authorities** จากการรับรองข้างต้น ดูเหมือนจะไม่มีปัญหาหากเครื่องของผู้รับบริการมีความปลอดภัย ไม่ถูกเพิ่มรายชื่อ Trusted Root Certification Authorities ใหม่แทรกเข้ามาได้ แต่หากพบปัญหา โดยมากมักถูกเพิ่มเข้ามาผ่านการติดตั้งซอฟต์แวร์ตัวอื่นๆ ภายหลัง ซึ่งมักจะเป็นจุดที่เจอได้ง่าย ตัวอย่างเช่น กรณี[ผู้ใช้โน้ตบุ๊ก Lenovo หลายรายพบว่าถูกติดตั้ง Adware มาจากโรงงาน, ดักฟังเว็บเข้ารหัส](https://www.blognone.com/node/65853 "ผู้ใช้โน้ตบุ๊ก Lenovo หลายรายพบว่าถูกติดตั้ง Adware มาจากโรงงาน, ดักฟังเว็บเข้ารหัส") แต่ยังมีอีกปัจจัยเสี่ยงคือ Trusted Root Certification Authorities เจ้าใดเจ้าหนึ่งในรายการที่เครื่องของผู้รับบริการถูกแฮก หรือเลินเล่อออกใบรับรองอิเล็กทรอนิกส์ของเว็บไซต์ซ้ำซ้อน จนทำให้มี TLS Certificate ที่เหมือนกับเว็บที่เราให้บริการ โดยเราที่เป็นผู้ให้บริการไม่ทราบ จึงทำให้การสื่อสารของบริการของเราสามารถถูกถอดรหัส และดักฟังได้โดยที่เรายังคงให้บริการได้เป็นปรกติ เหตุการณ์นี้ได้เกิดขึ้นแล้วกับ Google.com ผ่าน DigiNotar (รายละเอียดดูใน [แฮกเกอร์สร้างใบรับรอง SSL สำหรับโดเมนของกูเกิลได้สำเร็จ, ทุกเว็บของกูเกิลเสี่ยงต่อการถูกดักฟัง](https://www.blognone.com/news/26001/ "แฮกเกอร์สร้างใบรับรอง SSL สำหรับโดเมนของกูเกิลได้สำเร็จ, ทุกเว็บของกูเกิลเสี่ยงต่อการถูกดักฟัง")) หรือ [กูเกิลเตือนใบรับรองดิจิทัลปลอมจากอินเดีย](https://www.blognone.com/node/58174 "กูเกิลเตือนใบรับรองดิจิทัลปลอมจากอินเดีย") มาแล้ว [![Rogue-SSL-Certificate-Authority](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/Rogue-SSL-Certificate-Authority.png)](https://www.thaicyberpoint.com/ford/blog/id/5008/rogue-ssl-certificate-authority/#main)

รูปจาก [What is Certificate Transparency? How It helps Detect Fake SSL Certificates](http://thehackernews.com/2016/04/ssl-certificate-transparency.html)

**วิธีการต่างๆ ในการป้องกัน** จากเหตุการณ์ข้างต้น มีวิธีหลายวิธีในการป้องกันเหตุการณ์ดังกล่าว อย่าง [การทำ DNS-Based Authentication of Named Entities (DANE) บน TLSA record ที่ระดับ DNS ร่วมกับการทำ DNSSEC](https://www.thaicyberpoint.com/ford/blog/id/4519/) แต่มีความยุ่งยากที่ต้องติดตั้งและดูแลระบบ DNS ที่มีการปรับปรุงเพื่อให้รองรับ DNSSEC และ DANE เพิ่มมากขึ้น รวมไปถึง client ก็ยังไม่รองรับเป็นการทั่วไปสักเท่าไหร่ [![2016-06-10_020012](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/2016-06-10_020012-e1465498857208.png)](https://www.thaicyberpoint.com/ford/blog/id/5008/2016-06-10_020012/#main) 

แต่ก็ยังมีวิธีที่ง่ายกว่า และมีการรองรับโดยเบราว์เซอร์เจ้าใหญ่อย่าง Google Chrome และ Mozilla Firefox แล้ว คือ[HTTP Public Key Pinning (HPKP)](https://tools.ietf.org/html/rfc7469) เป็นกลไกที่ระบุว่าจะใช้ TLS Certificate ใดบ้าง ในการเชื่อมต่อเพื่อใช้งาน โดยจะรับรองผ่าน Certificate Authority เพียงเจ้าเดียว หรือหลายเจ้าก็ได้ **รู้จัก HTTP Public Key Pinning (HPKP)** ขั้นตอนการทำงานของ HPKP คือตรวจสอบจากค่าแฮชของ TLS Certificate นั้น ว่าตรงกับที่ระบุไว้ใน HTTP Header ในครั้งแรกที่ได้เข้าใช้บริการนั้นหรือไม่ (เทคนิคที่ว่าคือ [Trust on First Use](https://en.wikipedia.org/wiki/Trust_on_first_use) หรือ TOFU) เพราะหลังจากได้ข้อมูลดังกล่าวแล้ว client จะเชื่อค่าแฮชนั้นบนระยะเวลาที่กำหนดไว้ และจะอัพเดตค่าแฮชใหม่อีกครั้ง หลังจากที่ระยะเวลาดังกล่าวสิ้นสุดลง หากในระหว่างนั้นค่าแฮชของ TLS Certificate มีการเปลี่ยนแปลงไปจากที่รับรู้มาก่อนหน้านี้ จะไม่เชื่อถือ TLS Certificate ดังกล่าว 

[![2016-06-09_230144](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/2016-06-09_230144.png)](https://www.thaicyberpoint.com/ford/blog/id/5008/2016-06-09_230144/#main) 

วิธีการเปิดใช้งานคือ เพิ่ม header ที่ชื่อ _**Public-Key-Pins**_ หรือ _**Public-Key-Pins-Report-Only**_เมื่อมีการเข้าถึงผ่าน HTTPS โดยมีรูปแบบในตัวข้อมูลดังต่อไปนี้

    Public-Key-Pins: pin-sha256="base64=="; max-age=expireTime [; includeSubDomains][; report-uri="reportURI"]
    
    Public-Key-Pins-Report-Only: pin-sha256="base64=="; max-age=expireTime [; includeSubDomains][; report-uri="reportURI"]

อธิบายโค้ดแต่ละส่วน

*   **pin-sha256** _required_ ชุดตัวอักษรแบบ Base64 ซึ่งแปลงจากชุดตัวอักษรแฮชแบบ SHA-256 โดยข้อมูลดังกล่าวได้จาก Subject Public Key Information (SPKI) fingerprint ของไฟล์ Private Key, ไฟล์ Certificate Signing Request หรือไฟล์ Certificate (ที่ได้จาก Certificate Authority) _หมายเหตุ ในอนาคตอาจจะใช้ชุดตัวอักษรแฮชแบบอื่นๆ นอกจาก SHA-256 ก็ได้ ขึ้นอยู่กับมาตรฐานในอนาคต_
*   **max-age **_required_ ระยะเวลาที่บอก client ว่าจะจำค่าแฮชดังกล่าวไว้นานเท่าใดหน่วยเป็นวินาที
*   **includeSubDomains** _Optional_ เป็นชุดพารามิเตอร์ที่ไว้ระบุว่าค่าแฮชดังกล่าวรวมถึง subdomain ด้วย
*   **report-uri** _Optional_ เป็นชุดพารามิเตอร์ที่บอกว่าหากพบการตรวจสอบแล้วผิดพลาด ให้แจ้งไปยัง URL ที่ระบุไว้ โดยจะเป็นการ POST ชุดข้อมูลเป็น JSON ในรูปแบบต่อไปนี้
    
        {
             "date-time": date-time,
             "hostname": hostname,
             "port": port,
             "effective-expiration-date": expiration-date,
             "include-subdomains": include-subdomains,
             "noted-hostname": noted-hostname,
             "served-certificate-chain": [
               pem1, ... pemN
             ],
             "validated-certificate-chain": [
               pem1, ... pemN
             ],
             "known-pins": [
               known-pin1, ... known-pinN
             ]
        }
    
    ข้อมูลเพิ่มเติมอ่านต่อที่ [Reporting Pin Validation Failure](https://tools.ietf.org/html/rfc7469#page-16)

**จุดแตกต่างของ Public-Key-Pins และ Public-Key-Pins-Report-Only คือ** **Public-Key-Pins** จะตรวจสอบค่าแฮชของ TLS Certificate หากไม่ตรงตามที่กำหนดไว้จะไม่ยินยอมให้ client เข้าใช้งาน ถ้ามีการกำหนด report-uri ไว้ จะส่งรายงานไปยัง URL นั้น **Public-Key-Pins-Report-Only** จะตรวจสอบค่าแฮชของ TLS Certificate หากไม่ตรงตามที่กำหนดไว้จะรายงานไปยัง URL ที่กำหนดไว้ที่ report-uri แต่ยังยินยอมให้ client เข้าใช้งาน เมื่อทราบองค์ประกอบของตัวชุดข้อมูลบน HTTP header แล้ว ส่วนที่สำคัญคือชุดข้อมูล Base64 ที่จะใช้ใน pin-sha256 ซึ่งจะได้จากไฟล์ Private Key, ไฟล์ Certificate Signing Request หรือไฟล์ Certificate ชุดคำสั่งที่จะได้ค่าดังกล่าวมานั้นมีดังนี้ 1. ดึงค่า Base64 จาก Private Key

    openssl rsa -in private.key -outform der -pubout | openssl dgst -sha256 -binary | openssl enc -base64

2. ดึงค่า Base64 จาก Certificate Signing Request

    openssl req -in cert-signing-request.csr -pubkey -noout | openssl rsa -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64

3. ดึงค่า Base64 จาก Certificate

    openssl x509 -in certificate.crt -pubkey -noout | openssl rsa -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64

ตัวอย่างเมื่อรันคำสั่งตามข้างต้น จะได้ผลดังภาพด้านล่าง 

[![2016-06-10_003753](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/2016-06-10_003753.png)](https://www.thaicyberpoint.com/ford/blog/id/5008/2016-06-10_003753/#main)

ค่าที่ได้จะต้องเหมือนกันทั้ง 3 คำสั่ง แล้วนำเอาข้อมูลที่ได้ไปใส่ลงใน HTTP Header ปรกติ ก่อนใช้ Public-Key-Pins มักจะทดสอบด้วย Public-Key-Pins-Report-Only ก่อน เพื่อไม่ให้เกิดการ block การใช้งานเกิดขึ้นหากกำหนดค่า pin-sha256 ผิดพลาด เมื่อทดสอบเสร็จแล้วจึงกำหนดค่า Public-Key-Pins อีกครั้ง แต่บางคนอาจจะกำหนด Public-Key-Pins แล้วปรับ max-age ไว้ไม่เยอะก็ได้ (ตรงนี้แล้วแต่สะดวก) แล้วจึงกำหนด max-age ที่มีระยะเวลายาวนาน 1-2 เดือน เป็นอย่างน้อยที่สุด

    Public-Key-Pins: max-age=5184000; 
    pin-sha256="C8XLmj2eoYs1gqjNksWfsIA89sIAKNraGspBaXU/l+4="; 
    report-uri="https://thaicyberpoint.report-uri.io/r/default/hpkp/enforce"
    
    Public-Key-Pins-Report-Only: max-age=5184000; 
    pin-sha256="C8XLmj2eoYs1gqjNksWfsIA89sIAKNraGspBaXU/l+4="; 
    report-uri="https://thaicyberpoint.report-uri.io/r/default/hpkp/reportOnly"
    

คำแนะนำเพิ่มเติม การมีค่า pin-sha256 เพียงค่าเดียวกัน อาจจะมีปัญหาในช่วงที่ Certificate กำลังจะหมดอายุ หรือ Private Key ชุดเก่าหลุดจนเกิดปัญหา ทำให้ต้องมีการใช้ Private Key ชุดใหม่ เราจึงควรสร้าง Private Key สำรองเพื่อใช้สำหรับต่ออายุ Certificate หรือเหตุการณ์ที่ไม่คาดคิดในอนาคตไปสัก 2-3 ชุด แล้วใส่ค่า Base64 ของ Private Key ชุดดังกล่าวที่ยังไม่ได้ถูกใช้งานลงไปพร้อมกันด้วย

    Public-Key-Pins: pin-sha256="pzel6nQW2KK7geOhEeaci3qEDf1TJeg7CNCFqU9RvkQ=";
    pin-sha256="IlsUjvSKCiDJYyuVTifR07/qaIKCUn0ZK8z9mG1rAZ0=";
    pin-sha256="Rlef6668r6l11a8M2Ahf46PqK0Q4q+vnk/aRUCU+PeM="; 
    max-age=5184000;
    report-uri="https://thaicyberpoint.report-uri.io/r/default/hpkp/enforce"
    
    Public-Key-Pins-Report-Only: pin-sha256="pzel6nQW2KK7geOhEeaci3qEDf1TJeg7CNCFqU9RvkQ=";
    pin-sha256="IlsUjvSKCiDJYyuVTifR07/qaIKCUn0ZK8z9mG1rAZ0=";
    pin-sha256="Rlef6668r6l11a8M2Ahf46PqK0Q4q+vnk/aRUCU+PeM="; 
    max-age=5184000;
    report-uri="https://thaicyberpoint.report-uri.io/r/default/hpkp/reportOnly"
    

**การใช้งาน HPHK ในเว็บเซิร์ฟเวอร์** พอได้ข้อมูลครบตามนี้ ก็เอาไปใส่ในเว็บเซิร์ฟเวอร์ โดยบทความจะยกตัวอย่าง 3 ตัว คือ Apache, Nginx และ IIS โดยขออ้างอิงเพียง Public-Key-Pins ส่วน Public-Key-Pins-Report-Only ก็ใส่ไปคล้ายๆ กัน ต่างกันแค่ชื่อเริ่มต้นเท่านั้น

1.  **Apache** - (ต้องเปิด mod\_headers ด้วย) เพิ่มในส่วนของ Web Server config อาจจะใส่ในส่วนของ vhost ก็ได้
    
        Header always set Public-Key-Pins "pin-sha256=\"pzel6nQW2KK7geOhEeaci3qEDf1TJeg7CNCFqU9RvkQ=\";pin-sha256=\"IlsUjvSKCiDJYyuVTifR07/qaIKCUn0ZK8z9mG1rAZ0=\";pin-sha256=\"Rlef6668r6l11a8M2Ahf46PqK0Q4q+vnk/aRUCU+PeM=\"; max-age=5184000;report-uri=\"https://thaicyberpoint.report-uri.io/r/default/hpkp/enforce\""
    
2.  **Nginx **\- (ต้องเปิด ngx\_http\_headers\_module ด้วย) เพิ่มในส่วนของ Web Server config อาจจะใส่ในส่วนของ vhost ก็ได้
    
        add_header Public-Key-Pins 'pin-sha256="pzel6nQW2KK7geOhEeaci3qEDf1TJeg7CNCFqU9RvkQ=";pin-sha256="IlsUjvSKCiDJYyuVTifR07/qaIKCUn0ZK8z9mG1rAZ0=";pin-sha256="Rlef6668r6l11a8M2Ahf46PqK0Q4q+vnk/aRUCU+PeM=";max-age=5184000;report-uri="https://thaicyberpoint.report-uri.io/r/default/hpkp/enforce"';
    
3.  **IIS **\- แก้ไขที่ Web.config
    
        <httpProtocol>
         <customHeaders>
         <add name="Public-Key-Pins" value="pin-sha256=&quot;pzel6nQW2KK7geOhEeaci3qEDf1TJeg7CNCFqU9RvkQ=&quot;;pin-sha256=&quot;IlsUjvSKCiDJYyuVTifR07/qaIKCUn0ZK8z9mG1rAZ0=&quot;;pin-sha256=&quot;Rlef6668r6l11a8M2Ahf46PqK0Q4q+vnk/aRUCU+PeM=&quot;;max-age=5184000;report-uri=&quot;https://thaicyberpoint.report-uri.io/r/default/hpkp/enforce&quot;" />
         </customHeaders>
        </httpProtocol>
    

เมื่อตั้งค่าต่างๆ เรียบร้อยแล้ว ลองทดสอบผ่านทาง [SSL Server Test](https://www.ssllabs.com/ssltest/) เพื่อดูผลว่าตัวทดสอบสามารถตรวจจับข้อมูลที่เราได้ตั้งค่าได้หรือไม่ 

[![2016-06-10_013750](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/2016-06-10_013750.png)](https://www.thaicyberpoint.com/ford/blog/id/5008/2016-06-10_013750/#main) [![2016-06-10_150936](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/2016-06-10_150936.png)](https://www.thaicyberpoint.com/ford/blog/id/5008/2016-06-10_150936/#main) 

โดยหากเจอปัญหาค่าแฮชที่ตรวจสอบไม่ตรงตามขั้นตอนข้างต้น จะขึ้นข้อความตามด้านล่างนี้ บน Google Chrome และ Mozilla Firefox (อาจจะเปลี่ยนแปลงไปบ้างในแต่ละรุ่นที่ออกมาในอนาคต)

 [![hpkp_test_chrome](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/hpkp_test_chrome-e1465499618783.png)](https://www.thaicyberpoint.com/ford/blog/id/5008/hpkp_test_chrome/#main)

รูปจาก [What is Public Key Pinning / HPKP ? ](https://thomasschweizer.net/what-is-public-key-pinning-hpkp/)

[![](https://www.thaicyberpoint.com/ford/blog/wp-content/filesuploaded/2016/06/pin-validation-failure-e1465499764297.png)](https://www.thaicyberpoint.com/ford/blog/id/5008/pin-validation-failure/#main)

รูปจาก [About Public Key Pinning ](https://noncombatant.org/2015/05/01/about-http-public-key-pinning/)

ส่วนในกรณีที่ใช้การเขียนซอฟต์แวร์สำหรับเชื่อมต่อผ่าน API นั้น เราสามารถใช้การตรวจสอบในลักษณะนี้ ในการตรวจสอบความน่าเชื่อถือของการเชื่อมต่อได้ด้วยการเขียนชุดคำสั่งตรวจสอบได้เช่นกัน ตัวอย่างเช่น [Designing Evolvable Web APIs with ASP.NET – Chapter 15. Security](http://chimera.labs.oreilly.com/books/1234000001708/ch15.html) โดยเราจะใช้ลักษณะ Trust on First Use ตามแบบเว็บเบราว์เซอร์ หรือจะใช้การบันทึกค่าแฮชของ TLS Certificate ไว้ในตัวซอฟต์แวร์ก็ได้ เพราะการพัฒนาซอฟต์แวร์ที่เราควบคุมสภาพแวดล้อมได้ หากบันทึกค่าแฮชไว้ภายในตัวซอฟต์แวร์ก็จะช่วยลดความเสี่ยงในเรื่อง Trust on First Use ได้ **อะไรคือความเสี่ยงของ Trust on First Use?** คำตอบคือ หากในครั้งแรกที่เข้าใช้บริการ เครื่องที่เข้าใช้งานได้รับข้อมูลที่เป็นค่าแฮชปลอมบน HTTP Header ไปก่อนแล้ว วิธีการนี้ก็จะไม่สามารถตรวจสอบได้อีกต่อไปว่าจะเชื่อค่าแฮชชุดใดกันแน่ที่เป็นค่าแฮชจริง การใช้เทคนิค Trust on First Use จึงเป็นเพียงแค่ช่วยเพิ่มความสะดวกสำหรับคนใช้งานเบราว์เซอร์ทั่วไป สุดท้าย ในปัจจุบัน (ณ วันที่ 10/6/2016) มีเบราว์เซอร์อย่าง Google Chrome และ Mozilla Firefox รองรับ และหวังว่าจะมีเบราว์เซอร์ตัวอื่นๆ ในอนาคตที่จะเพิ่มเติมการตรวจสอบนี้เข้ามาในตัวเบราว์เซอร์ น่าจะช่วยให้เราให้บริการกับลูกค้าได้ปลอดภัยมากขึ้น รวมไปถึงซอฟต์แวร์ต่างๆ ที่พัฒนาจะใช้การจดจำค่าแฮชเหล่านี้เป็นค่าเริ่มต้น เพื่อช่วยป้องกัน และควบคุมการใช้ TLS Certificate ที่เราเป็นผู้รับผิดชอบเองจริงๆ ไม่ใช่เกิดจากการสวมรอย หรือออก TLS Certificate ที่ผิดพลาดจาก Certificate Authority ที่เราไม่ได้ใช้บริการอยู่ **อ้างอิงจาก**

*   [Public Key Pinning Extension for HTTP](https://tools.ietf.org/html/rfc7469) - Internet Engineering Task Force (IETF)
*   [Public Key Pinning](https://developer.mozilla.org/en-US/docs/Web/Security/Public_Key_Pinning) - MDN
*   [What is Certificate Transparency? How It helps Detect Fake SSL Certificates](http://thehackernews.com/2016/04/ssl-certificate-transparency.html) - The Hacker News