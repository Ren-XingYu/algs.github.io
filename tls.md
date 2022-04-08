https://www.barretlee.com/blog/2016/04/24/detail-about-ca-and-certs/

https://futu.im/article/https-certificate-1/

https://futu.im/article/https-certificate-2/

# 公钥&私钥

```
# 生成私钥
openssl genrsa -out grafana.key 2048
# 查看私钥
openssl rsa -in grafana.key -text -noout

# 生成公钥
openssl rsa -in grafana.key -pubout -out grafana.pem
# 查看公钥
openssl rsa -pubin -in grafana.pem -text -noout

# 签名请求
openssl req -new -subj "/C=CN/ST=GuangDong/L=ShenZhen/O=PMS/OU=OSS & Service Tools Dept/CN=10.136.194.167" -key grafana.key -out grafana.csr
# 查看签名请求
openssl req -in grafana.csr -text -noout

# 签发证书
openssl x509 -req -in grafana.csr -CA ca.cer -CAkey ca_key_no.pem -CAcreateserial -out grafana.crt -days 365 -extfile grafana.ext
# 查看证书
openssl x509 -in  grafana.crt -text -noout


openssl genrsa -out server.key 2048
openssl rsa -in grafana.key -text -noout 
openssl rsa -in server.key -pubout -out server.pem
```

# 自签证书（CA）

```
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -out ca.csr
openssl req -in ssltest.csr -noout -text

openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
openssl x509 -inform der -in futu.im.crt -noout -text
```

# 签发证书

```
openssl req -new -key server.key -out server.csr
openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt

/C=CN/ST=GuangDong/L=ShenZhen/O=PMS Technologies Co., Ltd/OU=OSS & Service Tools Dept/CN=OSS3.0 CA/emailAddress=defaultca
```

# 自建Root CA

**Root CA**

```
cd /root/ca
mkdir certs crl newcerts private
chmod 700 private
touch index.txt
echo 1000 > serial
wget -O /root/ca/openssl.cnf https://raw.githubusercontent.com/barretlee/autocreate-ca/master/cnf/root-ca

cd /root/ca
openssl genrsa -aes256 -out private/ca.key.pem 4096

cd /root/ca
openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem

openssl x509 -in certs/ca.cert.pem -text -noout
```

#### Intermediate CA

```
mkdir /root/ca/intermediate
cd /root/ca/intermediate
mkdir certs crl csr newcerts private
chmod 700 private
touch index.txt
echo 1000 > serial
echo 1000 > /root/ca/intermediate/crlnumber
wget -O /root/ca/intermediate/openssl.cnf https://raw.githubusercontent.com/barretlee/autocreate-ca/master/cnf/intermediate-ca

cd /root/ca
openssl genrsa -aes256 -out intermediate/private/intermediate.key.pem 4096
chmod 400 intermediate/private/intermediate.key.pem

cd /root/ca
openssl req -config intermediate/openssl.cnf -new -sha256 -key intermediate/private/intermediate.key.pem -out intermediate/csr/intermediate.csr.pem

cd /root/ca
openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem -out intermediate/certs/intermediate.cert.pem
# 此时root的index.txt中将会多一条记录,记录签发的证书

openssl x509 -in intermediate/certs/intermediate.cert.pem -text -noout
openssl verify -CAfile certs/ca.cert.pem intermediate/certs/intermediate.cert.pem

# 创建证书链
cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem
chmod 444 intermediate/certs/ca-chain.cert.pem

# 创建服务器证书
cd /root/ca
openssl genrsa -aes256 -out intermediate/private/www.barretlee.com.key.pem 2048
chmod 400 intermediate/private/www.barretlee.com.key.pem

cd /root/ca
openssl req -config intermediate/openssl.cnf -key intermediate/private/www.barretlee.com.key.pem -new -sha256 -out intermediate/csr/www.barretlee.com.csr.pem

cd /root/ca
openssl ca -config intermediate/openssl.cnf -extensions server_cert -days 375 -notext -md sha256 -in intermediate/csr/www.barretlee.com.csr.pem -out intermediate/certs/www.barretlee.com.cert.pem
chmod 444 intermediate/certs/www.barretlee.com.cert.pem
# 可以看到 /root/ca/intermediate/index.txt 中多了一条记录：

```

```
openssl x509 -noout -text -in intermediate/certs/www.barretlee.com.cert.pem
openssl verify -CAfile intermediate/certs/ca-chain.cert.pem intermediate/certs/www.barretlee.com.cert.pem
```

```
签发CA证书和终端证书有两处不同：
1、生成证书请求文件的时候。读者可查看openssl.cnf中[req]字段中扩展字段是v3_req，在v3_req中有个basicConstraints变量，
     当basicConstraints=CA:TRUE时，表明要生成的证书请求是CA证书请求文件；
     当basicConstraints=CA:FALSE时，表明要生成的证书请求文件是终端证书请求文件;
2、在签发证书的时候。签发终端证书的时候使用默认扩展字段usr_cert，当签发CA证书的时候再命令行使用了extensions选项指定v3_ca字段。
     在默认的usr_cert字段中 basicConstraints=CA:FALSE；表明要签发终端证书
     而在v3_ca字段中 basicConstraints=CA:TRUE；表明要签发CA证书
```





```
openssl version
	1.0.1 支持TLS1.1、TLS1.2
	1.1.1 支持TLS1.1、TLS1.2、TLS1.3
openssl version -a
	OPENSSLDIR
		misc(miscellaneous)	
openssl help
```

# Key Generation

```
常用算法：RSA(常用)、DSA(obsolete)、ECDSA(常用)、EdDSA
```

## 老的命令

### **RSA**

512bit、1024bit、2048bit、4096bit

```
openssl genrsa -aes256 -out fd.key 2048	# PEM格式存储
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,F6DC1A26828CE2F0682FE3097912DF20

xxxxxx...
-----END RSA PRIVATE KEY-----

openssl rsa -text -noout -in fd.key 

openssl rsa -in fd.key -pubout -out fd-public.key
-----BEGIN PUBLIC KEY-----
xxxxxx...
-----END PUBLIC KEY-----
```

### **DSA**

512bit、1024bit、2048bit、4096bit

```
openssl dsaparam -genkey 2048 | openssl dsa -out dsa.key -aes256
-----BEGIN DSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,F376A08BE7414E93DEB435F4861FE846

xxxxxx...
-----END DSA PRIVATE KEY-----

openssl dsa -text -noout -in dsa.key

openssl dsa -in dsa.key -pubout -out dsa-public.key
-----BEGIN PUBLIC KEY-----
xxxxxx...
-----END PUBLIC KEY-----
```

### **ECDSA**

```
# 浏览器支持两种主流的曲线： secp256r1 (OpenSSL uses the name prime256v1) and secp384r1.

# 查看支持的曲线
openssl ecparam -list_curves

openssl ecparam -genkey -name secp256r1 | openssl ec -out ec.key -aes256
-----BEGIN EC PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,504E332F00FA0907993941147CE8E891

xxxxxx...
-----END EC PRIVATE KEY-----

openssl ec -text -noout -in ec.key

openssl ec -in ec.key -pubout -out ec-public.key
-----BEGIN PUBLIC KEY-----
xxxxxx...
-----END PUBLIC KEY-----
```

## 统一后的命令

### **RSA**

```
openssl genpkey -out fd.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -aes-256-cbc	# PKCS#8格式存储
-----BEGIN ENCRYPTED PRIVATE KEY-----
xxxxxx...
-----END ENCRYPTED PRIVATE KEY-----

openssl pkey -in fd.key -text -noout

openssl pkey -in fd.key -pubout -out fd-public.key
-----BEGIN PUBLIC KEY-----
xxxxxx...
-----END PUBLIC KEY-----
```

### **ECDSA**

```
# Web浏览器支持两种曲线
P-256 (also known as secp256r1 or prime256v1)：已经足够安全，提供能耗的性能
P-384 (secp384r1)

# 查看支持的曲线
openssl ecparam -list_curves

openssl genpkey -out fd.key -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -aes-256-cbc
-----BEGIN ENCRYPTED PRIVATE KEY-----
xxxxxx...
-----END ENCRYPTED PRIVATE KEY-----

openssl pkey -in fd.key -text -noout

openssl pkey -in fd.key -pubout -out ec-public.key
-----BEGIN PUBLIC KEY-----
xxxxxx...
-----END PUBLIC KEY-----
```

# Certificate Signing Requests

```
openssl req -new -key fd.key -out fd.csr

openssl req -in fd.csr -text -noout

# 从已有的证书提取CSR
openssl x509 -x509toreq -in fd.crt -out fd.csr -signkey fd.key
```

**fd.cnf**

```
[req]
prompt = no
distinguished_name = dn
req_extensions = ext
input_password = PASSPHRASE

[dn]
CN = www.feistyduck.com
emailAddress = webmaster@feistyduck.com
Signing Your Own Certificates
15
O = Feisty Duck Ltd
L = London
C = GB

[ext]
subjectAltName = DNS:www.feistyduck.com,DNS:feistyduck.com
```

```
openssl req -new -config fd.cnf -key fd.key -out fd.csr
```

# Signing Your Own Certificates

```
openssl x509 -req -days 365 -in fd.csr -signkey fd.key -out fd.crt

# 不创建CSR文件,直接生成证书
openssl req -new -x509 -days 365 -key fd.key -out fd.crt

openssl req -new -x509 -days 365 -key fd.key -out fd.crt -subj "/C=GB/L=London/O=Feisty Duck Ltd/CN=www.feistyduck.com"
```

# Creating Certificates Valid for Multiple Hostnames

There are two mechanisms for supporting multiple hostnames in a certificate. The first is to list all desired hostnames using an X.509 extension called **Subject Alternative Name (SAN)**.The second is to use wildcards. You can also use a combination of the two approaches when it’s more convenient. In practice, for most sites, you can specify a bare domain name and a wildcard to cover all the subdomains (e.g., *feistyduck.com* and **.feistyduck.com*).

When a certificate contains alternative names, all common names are ignored. Newer certificates produced by CAs may not even include any common names. For that reason, include all desired hostnames on the alternative names list.

The Subject Alternative Name extension is used to list all the hostnames for which the certificate is valid. This extension used to be optional; if it isn’t present, clients fall back to using the information provided in the common name (CN), which is part of the Subject field.If the extension is present, then the content of the CN field is ignored during validation.

**fd.ext**

```
subjectAltName = DNS:*.feistyduck.com, DNS:feistyduck.com
```

```
openssl x509 -req -days 365 -in fd.csr -signkey fd.key -out fd.crt -extfile fd.ext
```

# Examining Certificates

```
openssl x509 -text -in fd.crt -noout
```

