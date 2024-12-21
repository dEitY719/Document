관리자 권한으로 CMD 실행

cd C:\Program Files\Microsoft Office\Office16
for /f %x in ('dir /b ..\root\Licenses16\ProPlus2021VL*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%x"
cscript ospp.vbs /setprt:1688
cscript ospp.vbs /unpkey:6MWKP >nul
cscript ospp.vbs /inpkey:NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP
cscript ospp.vbs /sethst:kms8.msguides.com
cscript ospp.vbs /act


출처: https://lucky-dreams.tistory.com/110
