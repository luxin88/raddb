作为学习或测试使用，我们当然不需要花钱去权威机构申请证书，只要自己做一个就可以了。OpenSSL可以帮你完成这项工作。
1、先做一张CA证书，有了它，你就是所谓的权威机构，你就可以给别人签发证书了。
§openssl req -new -x509 -keyout ca.key -out ca.crt -config openssl.cnf 
这里有两个输出文件（ca.key和ca.crt）和一个输入文件（openssl.cnf）。ca.crt就是ca证书了，ca.key就是ca证书的私钥，不过是经过加密的了。openssl.cnf就是openssl的config文件了。值得说明的是，还有几个文件是所必须的，先罗列一下。文件：index.txt，dh，index.txt.attr，random，serial；目录：certs，private，crl，newcerts。
2、为用户生成私钥
§openssl genrsa -des3 -out client.key 1024
des3是一种对称加密方式，这里是对生成的私钥进行加密的。当然你也可以采用其它的方式进行加密，如，aes等。
3、为用户生成证书请求文件
openssl req -new -key client.key -out client.csr -config openssl.cnf 
4、使用CA证书对用户的证书请求文件进行签署，生成用户证书
openssl ca -in client.csr -out client.crt -cert ca.crt -keyfile ca.key -config openssl.cnf 
至此，用户的证书已经生成完毕。通常情况下，对于ms-window的客户端来说，证书都是采用pkcs12的格式，这样更容易安装。
5、生成pkcs12格式的证书文件
openssl pkcs12 –export –inkey client.key –in client.crt –out client.p12
openssl pkcs12 –export –inkey ca.key –in ca.crt –out ca.p12
这样，你就可以将client.p12和ca.p12拿到你的window机器上安装了，安装方式很简单，只要双击就可以了，按提示进行。ca.p12要放到受信任的根证书下。

对于Freeradius来说，也需要证书，我暂且把它叫做server证书。其生成方式和用户证书一样。（其实只是取的名字不一样，原理都一样）。需要说明的是，Freeradius一般识别的是pem类型文件，所以将私钥和证书合并成一个pem文件。
6、生成Freeradius需要的pem文件
cat ca.crt ca.key > ca.pem 
cat server.crt server.key > server.pem
ca.pem和server.pem就是Freeradius所需要的了，完毕。

四、Freeradius的EAP配置
其实主要是指定证书的位置就好了。
tls {                     
  certdir = /usr/local/etc/raddb/cert_kun
  cadir = /usr/local/etc/raddb/cert_kun
  private_key_password = trapeze
  private_key_file = ${certdir}/server.pem
  CA_file = ${cadir}/ca.pem
  dh_file = ${certdir}/dh
  random_file = ${certdir}/random
  ....
}

