# แนะนำการใช้ HTTP Strict Transport Security (HSTS) เพื่อให้เว็บเบราว์เซอร์ติดต่อผ่านช่องทางเข้ารหัสเท่านั้น

HTTP Strict Transport Security (ต่อไปจะเรียกว่า HSTS) เป็นการเพิ่มประสิทธิภาพในรักษาความปลอดภัย เพื่อบอกให้เว็บเบราว์เซอร์ที่กำลังเข้ามาใช้บริการเว็บไซต์ ต้องทำงานผ่านช่องทางเข้ารหัส Hypertext Transfer Protocol Secure เท่านั้น (ต่อไปจะเรียกว่า HTTPS)

โดยจะเพิ่มชุดคำสั่งด้านล่างลงใน HTTP Header เพื่อส่งไปบอกเว็บเบราว์เซอร์

    Strict-Transport-Security: max-age=expireTime [; includeSubDomains][; preload]

**อธิบายโค้ดแต่ละส่วน**

*   **max-age** _required_  
    ระยะเวลาที่บอก client ว่าจะให้จำว่าควรเข้าเว็บผ่านช่องทางเข้ารหัส HTTPS นานเท่าใด โดยหน่วยเป็นวินาที
*   **includeSubDomains** _Optional_  
    ระบุว่าการใช้งาน HSTS ดังกล่าวรวมถึง subdomain ด้วย
*   **preload** _Optional_  
    หากในตัว domain ข้างต้น มีการเรียกใช้ข้อมูลใด ๆ ที่โหลดมาจาก domain ที่อยู่ในรายการ HSTS preload service บน browser ให้ใช้การเชื่อมต่อผ่านช่องทางเข้ารหัส HTTPS เช่นกัน โดย HSTS preload service นี้ Google เป็นคนดูแลรายชื่อ domain ดังกล่าวผ่าน [](https://hstspreload.org)[https://hstspreload.org](https://hstspreload.org) ซึ่งตอนนี้มี Browser อย่าง Chrome, Firefox, Opera, Safari, IE 11 และ Edge ที่ใช้รายการ domain ดังกล่าวใส่ลงใน browser ตัวเอง ซึ่งมักถูกแนะนำให้ใช้เสมอหากเราใช้ 3rd party อย่าง Javascript หรือ CSS ที่โหลดผ่าน CDN ต่าง ๆ

**การใช้งาน HSTS ในเว็บเซิร์ฟเวอร์**

พอได้ข้อมูลครบตามนี้ ก็เอาไปใส่ในเว็บเซิร์ฟเวอร์ โดยบทความจะยกตัวอย่าง 4 ตัว คือ Apache, Nginx, IIS และ .NET Core

**Apache** – (ต้องเปิด mod\_headers ด้วย) เพิ่มในส่วนของ Web Server config อาจจะใส่ในส่วนของ vhost ก็ได้

    Header always set Strict-Transport-Security: max-age=31536000

หรือหากมีเพิ่มพารามิเตอร์

    Header always set Strict-Transport-Security: max-age=31536000; includeSubDomains

**Nginx** – (ต้องเปิด ngx\_http\_headers\_module ด้วย) เพิ่มในส่วนของ Web Server config อาจจะใส่ในส่วนของ vhost ก็ได้

    add_header Strict-Transport-Security: max-age=31536000

หรือหากมีเพิ่มพารามิเตอร์

    add_header Strict-Transport-Security: max-age=31536000; includeSubDomains

**IIS** – แก้ไขที่ Web.config

    <httpProtocol>
     <customHeaders>
     <add name="Strict-Transport-Security" value="max-age=31536000" />
     </customHeaders>
    </httpProtocol>

หรือหากมีเพิ่มพารามิเตอร์

    <httpProtocol>
     <customHeaders>
     <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains" />
     </customHeaders>
    </httpProtocol>

**ASP.NET Core 2.1 หรือมากกว่า** เพิ่มเติมดังนี้

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        ...
        app.UseHsts();
        ...
    }

การแก้ไขพารามิเตอร์ทำผ่าน method ชื่อ ConfigureServices ได้ตามด้านล่าง

    public void ConfigureServices(IServiceCollection services)
    {
        ...
        services.AddHsts(options =>
        {
            options.Preload = true;
            options.IncludeSubDomains = true;
            options.MaxAge = TimeSpan.FromDays(365);
        });
        ...
    }

จากข้อมูลการใช้งาน และตัวอย่างทั้งหมด การเพิ่มเติมดังกล่าวมีประโยชน์ในการช่วยให้ผู้ใช้งาน เข้าเว็บผ่าน HTTPS ได้ปลอดภัยขึ้น หากในครั้งแรกที่เข้าเว็บได้รับ HTTP Header ข้างต้นอย่างถูกต้อง ในครั้งต่อไปตัวเว็บเบราว์เซอร์จะเข้าผ่าน URL protocal "https://" ทันที โดยไม่ต้องผ่าน "http://" อีก ทำให้ลดการถูก man-in-the-middle attacker เพื่อดังฟังข้อมูลอื่น ๆ ระหว่างการเปลี่ยนผ่านจาก URL protocal "http://" ไปยัง "https://" ได้ รวมไปถึงการโดน intercept traffic เพื่อให้เรารับ Certificate ที่ไม่ถูกต้องระหว่างการเข้าถึงหน้า HTTPS ได้

ไม่ได้เขียนเรื่องเทคโนโลยีมานาน รอบนี้เป็นบทความที่ดองมานานมาก รอบนี้เป็นบทความที่ต่อจากเรื่อง [HTTP Public Key Pinning (HPKP) เพื่อป้องกัน Certification Authority ออก TLS Certificate ซ้ำซ้อน](https://www.thaicyberpoint.com/ford/blog/id/5008/) ที่เขียนไปก่อนหน้านี้

โดย HTTP Strict Transport Security (ต่อไปจะเรียกว่า HSTS) เป็นการเพิ่มประสิทธิภาพในรักษาความปลอดภัย เพื่อบอกให้เว็บเบราว์เซอร์ที่กำลังเข้ามาใช้บริการเว็บไซต์ ต้องทำงานผ่านช่องทางเข้ารหัส Hypertext Transfer Protocol Secure เท่านั้น (ต่อไปจะเรียกว่า HTTPS)

โดยจะเพิ่มชุดคำสั่งด้านล่างลงใน HTTP Header เพื่อส่งไปบอกเว็บเบราว์เซอร์

    Strict-Transport-Security: max-age=expireTime [; includeSubDomains][; preload]

**อธิบายโค้ดแต่ละส่วน**

*   **max-age** _required_  
    ระยะเวลาที่บอก client ว่าจะให้จำว่าควรเข้าเว็บผ่านช่องทางเข้ารหัส HTTPS นานเท่าใด โดยหน่วยเป็นวินาที
*   **includeSubDomains** _Optional_  
    ระบุว่าการใช้งาน HSTS ดังกล่าวรวมถึง subdomain ด้วย
*   **preload** _Optional_  
    หากในตัว domain ข้างต้น มีการเรียกใช้ข้อมูลใด ๆ ที่โหลดมาจาก domain ที่อยู่ในรายการ HSTS preload service บน browser ให้ใช้การเชื่อมต่อผ่านช่องทางเข้ารหัส HTTPS เช่นกัน โดย HSTS preload service นี้ Google เป็นคนดูแลรายชื่อ domain ดังกล่าวผ่าน [](https://hstspreload.org)[https://hstspreload.org](https://hstspreload.org) ซึ่งตอนนี้มี Browser อย่าง Chrome, Firefox, Opera, Safari, IE 11 และ Edge ที่ใช้รายการ domain ดังกล่าวใส่ลงใน browser ตัวเอง ซึ่งมักถูกแนะนำให้ใช้เสมอหากเราใช้ 3rd party อย่าง Javascript หรือ CSS ที่โหลดผ่าน CDN ต่าง ๆ

**การใช้งาน HSTS ในเว็บเซิร์ฟเวอร์**

พอได้ข้อมูลครบตามนี้ ก็เอาไปใส่ในเว็บเซิร์ฟเวอร์ โดยบทความจะยกตัวอย่าง 4 ตัว คือ Apache, Nginx, IIS และ .NET Core

**Apache** – (ต้องเปิด mod_headers ด้วย) เพิ่มในส่วนของ Web Server config อาจจะใส่ในส่วนของ vhost ก็ได้

    Header always set Strict-Transport-Security: max-age=31536000

หรือหากมีเพิ่มพารามิเตอร์

    Header always set Strict-Transport-Security: max-age=31536000; includeSubDomains

**Nginx** – (ต้องเปิด ngx_http_headers_module ด้วย) เพิ่มในส่วนของ Web Server config อาจจะใส่ในส่วนของ vhost ก็ได้

    add_header Strict-Transport-Security: max-age=31536000

หรือหากมีเพิ่มพารามิเตอร์

    add_header Strict-Transport-Security: max-age=31536000; includeSubDomains

**IIS** – แก้ไขที่ Web.config

    <httpProtocol>
     <customHeaders>
     <add name="Strict-Transport-Security" value="max-age=31536000" />
     </customHeaders>
    </httpProtocol>

หรือหากมีเพิ่มพารามิเตอร์

    <httpProtocol>
     <customHeaders>
     <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains" />
     </customHeaders>
    </httpProtocol>

**ASP.NET Core 2.1 หรือมากกว่า** เพิ่มเติมดังนี้

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        ...
        app.UseHsts();
        ...
    }

การแก้ไขพารามิเตอร์ทำผ่าน method ชื่อ ConfigureServices ได้ตามด้านล่าง

    public void ConfigureServices(IServiceCollection services)
    {
        ...
        services.AddHsts(options =>
        {
            options.Preload = true;
            options.IncludeSubDomains = true;
            options.MaxAge = TimeSpan.FromDays(365);
        });
        ...
    }

จากข้อมูลการใช้งาน และตัวอย่างทั้งหมด การเพิ่มเติมดังกล่าวมีประโยชน์ในการช่วยให้ผู้ใช้งาน เข้าเว็บผ่าน HTTPS ได้ปลอดภัยขึ้น หากในครั้งแรกที่เข้าเว็บได้รับ HTTP Header ข้างต้นอย่างถูกต้อง ในครั้งต่อไปตัวเว็บเบราว์เซอร์จะเข้าผ่าน URL protocal “https://” ทันที โดยไม่ต้องผ่าน “http://” อีก ทำให้ลดการถูก man-in-the-middle attacker เพื่อดังฟังข้อมูลอื่น ๆ ระหว่างการเปลี่ยนผ่านจาก URL protocal “http://” ไปยัง “https://” ได้ รวมไปถึงการโดน intercept traffic เพื่อให้เรารับ Certificate ที่ไม่ถูกต้องระหว่างการเข้าถึงหน้า HTTPS ได้
