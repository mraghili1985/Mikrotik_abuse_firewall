# تنظیمات فایروال برای میکروتیک

## 1. افزودن رنج‌های IP خصوصی برای بلوک کردن

/ip firewall address-list
add list=RFC1918 address=10.0.0.0/8
add list=RFC1918 address=172.16.0.0/12
add list=RFC1918 address=192.168.0.0/16
add list=RFC1918 address=100.64.0.0/10

##2. بلوک کردن رنج‌های IP خصوصی از خروجی (جلوگیری از نشت)
/ip firewall filter
add chain=output dst-address-list=RFC1918 action=drop comment="Block private IP addresses from being routed outside"


##3. بلوک کردن آدرس‌های لینک محلی و آدرس‌های رزرو شده
/ip firewall filter
add action=drop chain=forward dst-address=169.254.0.0/16 comment="Block Link-local addresses"
add action=drop chain=forward dst-address=192.168.0.0/16 comment="Block local network traffic"
add action=drop chain=forward dst-address=100.64.0.0/10 comment="Block CGNAT range"

##4. بلوک کردن رنج آدرس‌های 172.0.0.0/8 (آدرس‌های عمومی غیرمجاز)
/ip firewall filter
add action=drop chain=forward dst-address=172.0.0.0/8 comment="Block range 172.0.0.0/8 (forbidden public address range)"


##5. بلوک کردن پورت‌های SMTP و تورنت از مشتریان VPN

/ip firewall filter
add chain=forward dst-port=25 protocol=tcp action=drop comment="Block SMTP outbound traffic"
add chain=forward dst-port=6881-6999 protocol=tcp action=drop comment="Block Torrent traffic (TCP)"

##6. بلوک کردن درخواست‌های DNS از منابع غیر محلی
/ip firewall filter
add chain=forward dst-port=53 protocol=udp action=drop comment="Block DNS request from non-local sources"


##7. محدود کردن اتصالات جدید برای جلوگیری از سوءاستفاده
/ip firewall filter
add chain=forward protocol=tcp connection-state=new connection-limit=30,32 action=drop comment="Limit excessive new connections per IP"

##8. شناسایی اسکنرهای پورت و اتصالات ورودی ناخواسته
/ip firewall filter
add chain=forward protocol=tcp psd=21,3s,3,1 action=add-src-to-address-list address-list=port-scanners address-list-timeout=1d comment="Detect port scanning attempts"
add chain=forward src-address-list=port-scanners action=drop comment="Drop connections from port scanners"
