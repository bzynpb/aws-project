
<a href="https://ibb.co/bPtq5WV"><img src="https://i.ibb.co/CzLNK1y/image.png" alt="image" border="0" /></a>

- *1  mapping ozelligini kullanabilmek icin; 
    ImageId: mapping kullanabilmek icin, region'a gore 2 kategori belirleyip oyle secicez (production ve test icin)
    imageID onceden olusturmamiz gerek ki mapping kisminda kullanabilelim
```
      myRegionImageMap:
    us-east-1:
      prod: ami-0453ec754f44f9a4a
      test: ami-0ed83e7a78a23014e
    us-east-2:
      prod: ami-0c80e2b6ccb9ad6d1
      test: ami-0a9f08a6603f3338e
    us-west-1:
      prod: ami-038bba9a164eb3dc1
      test: ami-0abe6f915c415296f
```

    dokumanda mapping kismini incele

- *2 LaunchTemplate image icinde test mi product mi consonle icinden secilmesi icin parametre yarat

- *3 instancetype icin de parametre olustur 

- *4 UserData: !Base64 |   # loadbalancer'da kullanilan hazir datayi kullaniyoruz 
```
#!/bin/bash
     #update os
     dnf update -y
     #install apache server
     dnf install -y httpd
     # get private ip address of ec2 instance using instance metadata
     TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` \
     && PRIVATE_IP=`curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/local-ipv4`
     # get public ip address of ec2 instance using instance metadata
     TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` \
     && PUBLIC_IP=`curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/public-ipv4`
     # get date and time of server
     DATE_TIME=`date`
     # set all permissions
     chmod -R 777 /var/www/html
     # create a custom index.html file
     echo "<html>
       <head>
       <title> Application Load Balancer</title>
       </head>
       <body>
         <h1>Testing Application Load Balancer</h1>
         <h2>Congratulations! You have created an instance from Launch Template</h2>
         <h3>This web server is launched from the launch template by BZB via CFN</h3>
         <p>This instance is created at <b>$DATE_TIME</b></p>
         <p>Private IP address of this instance is <b>$PRIVATE_IP</b></p>
         <p>Public IP address of this instance is <b>$PUBLIC_IP</b></p>
       </body>
       </html>" > /var/www/html/index.html
       # start apache server
       systemctl start httpd
       systemctl enable httpd

```

- Load balancer olustur
 (listener yaml formatini da eklemeyi unutma)

    - DefaultActions: zorunlu, inceleyerek neler yazabilecegine bak 
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-elasticloadbalancingv2-lis

template hazirlandiktan sonra:
    --> aws > cloudformation
    --> create stake