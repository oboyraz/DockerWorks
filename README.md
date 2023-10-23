# 1: İlk olarak sistemde bir temizlik yapalım ki alıştırmalarımızla çakışma olmasın. 
Varsa sistemdeki tüm containerları silelim. 

docker container ls -a

# 2: Docker logout ve docker login komutlarını kullanarak hesabımızdan logout olup tekrar login olalım. 

docker logout
docker login

# 3: Önceden oluşturduğunuz ve saklamanız gereken imajlar var ise bunları docker hub'a gönderin 
ve ardından sistemdeki tüm imajları silin

docker image push oboyraz/py

# 4: Docker hub'da kendi hesabınız altinda "alistirma" adıyla public bir repository yaratın. 



# 5: Centos imajının latest ve 7, ubuntu imajının latest, 18.04 ve 20.04, alpine imajının latest, 
nginx imajının latest ve alpine tagli imajlarını sisteme çekin. 

docker image pull centos:latest 
centos:7 ubuntu:latest ubuntu:18.04 ubuntu:20.04 alpine:latest nginx:latest nginx:alpine   

# 6: ubuntu:18.04 imajına dockerhubkullaniciadiniz/alistirma:ubuntu olarak tag ekleyin ve ardından bu yeni imajı docker hub'a gönderin. Alistirma repository'inizden imajı check edin. 

docker image tag ubuntu:18.04 oboyraz/alistirma:ubuntu

docker image push oboyraz/alistirma:ubuntu

# 7:Bu alistirma.txt dosyasının olduğu klasörde bir Dockerfile oluşturun: 
- Base imaj olarak nginx:latest imajını kullanın
- İmaja LABEL="kendi adınız ve erişim bilgileriniz" şeklinde label ekleyiniz. 
- KULLANICI adında bir enviroment variable tanımlayın ve değer olarak adınızı atayın
- RENK adından bir build ARG tanımlayın
- Sistemi update edin ve ardından curl, htop ve wget uygulamalarını kurun
- /gecici klasörüne geçin ve https://wordpress.org/latest.tar.gz dosyasını buraya ekleyin
- /usr/share/nginx/html klasörüne geçin ve html/${RENK}/ klasörünün içeriğini buraya kopyalayın
- Healtcheck talimatı girelim. curl ile localhost'u kontrol etsin. Başlangıç periodu 5 saniye, deneme aralığı 30s ve
zaman aşımı süresi de 30 saniye olsun. Deneme sayısı 3 olsun. 
- Bu imajdan bir container yaratıldığı zaman ./script.sh dosyasının çalışmasını sağlayan talimatı exec formunda girin. 


FROM nginx:latest
LABEL maintainer="Omer Boyraz oboyraz@sakarya.edu.tr"
ENV KULLANICI="Omer"
ARG RENK
RUN apt-get update && apt-get install curl -y && apt-get install htop -y && apt-get install wget -y
WORKDIR /gecici
ADD https://wordpress.org/latest.tar.gz .
WORKDIR /usr/share/nginx/html
COPY html/${RENK}/ .
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 CMD curl -f http://localhost/ || exit 1
CMD ["./script.sh"]


# 8: Bu Dockerfile dosyasından 2 imaj yaratın. Birinci imajda build ARG olarak RENK:kirmizi ikinci imajda da
build ARG olarak RENK:sari kullanın. Kirmizi olan imajın tagi dockerhubkullaniciadiniz/alistirma:kirmizi 
Sari olan imajin tagi dockerhubkullaniciadiniz/alistirma:sari olsun. 


docker build -t oboyraz/alistirma:kirmizi --build-arg RENK=kirmizi .

docker build -t oboyraz/alistirma:sari --build-arg RENK=sari .

# 9: dockerhubkullaniciadiniz/alistirma:kirmizi imajını kullanarak bir container yaratın. Detach olsun.
Makinenin 80 portuna gelen istekler bu containerın 80 portuna gitsin. Container adi kirmizi olsun.
Browser'dan http://127.0.0.1 sayfasına gidip check edin.  

docker container run -d -p 80:80 --name kirmizi oboyraz/alistirma:kirmizi

# 10: dockerhubkullaniciadiniz/alistirma:sari imajını kullanarak bir container yaratın. Detach olsun.
Makinenin 8080 portuna gelen istekler bu containerın 80 portuna gitsin.
KULLANICI enviroment variable değerini "Deneme" olarak atayın. Container adi sari olsun. 
Browser'dan http://127.0.0.1:8080 sayfasına gidip check edin.

docker container run -d -p 8080:80 --name sari --env KULLANICI=Deneme oboyraz/alistirma:sari

# 11: Containerları kapatıp silelim. 

# 12: Bu alistirma.txt dosyasının olduğu klasörde Dockerfile.multi isimli bir Dockerfile oluşturun: 
- Bu multi stage build alıştırması olacak. 
- Birinci stage'i  mcr.microsoft.com/java/jdk:8-zulu-alpine imajından oluşturun ve stage adı birinci olsun
- /usr/src/uygulama klasörüne geçin ve source klasörünün içeriğini buraya kopyalayın
- "javac uygulama.java" komutunu çalıştırarak uygulamanızı derleyin
- mcr.microsoft.com/java/jre:8-zulu-alpine imajından ikinci aşamayı başlatın. 
- /uygulama klasörüne geçin ve birinci aşamadıki imajın /usr/src/uygulama klasörünün içeriğini buraya kopyalayın
- Bu imajdan container yaratıldığı zaman "java uygulama" komutunun çalışması için talimat girin


FROM mcr.microsoft.com/java/jdk:8-zulu-alpine AS birinci
WORKDIR /usr/src/uygulama
COPY /source .
RUN javac uygulama.java

FROM mcr.microsoft.com/java/jre:8-zulu-alpine
WORKDIR /uygulama
COPY --from=birinci /usr/src/uygulama .
CMD ["java", "uygulama"]


# 13: Bu Dockerfile.multi dosyasından dockerhubkullaniciadiniz/alistirma:java tagli bir imaj yaratın. 

docker build -t oboyraz/alistirma:java -f Dockerfile.multi .


# 14: Bu imajdan bir container yaratın ve java uygulamanızın çıkardığı mesajı görün.

docker container run --name javauygulama oboyraz/alistirma:java

# 15: dockerhubkullaniciadiniz/alistirma:kirmizi, dockerhubkullaniciadiniz/alistirma:sari, dockerhubkullaniciadiniz/alistirma:java imajlarını Docker hub'a yollayın. 

docker image push oboyraz/alistirma:kirmizi 

docker image push oboyraz/alistirma:sari

docker image push oboyraz/alistirma:java

# 16: Docker hub'daki registry isimli imajdan lokal bir Docker Registry çalıştırın. 

docker pull registry

docker run -d -p 5000:5000 --restart always --name registry registry


# 17: dockerhubkullaniciadiniz/alistirma:kirmizi, dockerhubkullaniciadiniz/alistirma:sari, dockerhubkullaniciadiniz/alistirma:java imajlarını yeniden tagleyerek bu lokal registry'e gönderin ve ardından bu registry'nin web arayüzünden kontrol edin. 
