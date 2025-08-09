
# Automatizando Snapshots e Gerenciando Buckets no AWS S3 e EC2

Este laboratório utiliza AWS EBS para criação de snapshots automatizados e Amazon S3 para gerenciamento de buckets, incluindo controle de versão e restauração de arquivos.



## 1️⃣ Criando e configurando o bucket no Amazon S3
Criar um bucket no Amazon S3.

Na instância EC2 Processor, permitir acesso ao bucket:

Actions → Security → Modify IAM Role → conceder acesso ao bucket.

2️⃣ Conectando ao Command Host
Acessar via Instance Connect.

3️⃣ Coletando IDs necessários
Volume ID

aws ec2 describe-instances --filter 'Name=tag:Name,Values=Processor' \
--query 'Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.{VolumeId:VolumeId}'

Exemplo de retorno:
"VolumeId": "vol-051f64d8fb1d27d2d"

Instance ID

aws ec2 describe-instances --filters 'Name=tag:Name,Values=Processor' \
--query 'Reservations[0].Instances[0].InstanceId'

Exemplo de retorno:
"i-045ab79b3f920f54e"

4️⃣ Parando a instância

aws ec2 stop-instances --instance-ids i-045ab79b3f920f54e
aws ec2 wait instance-stopped --instance-id i-045ab79b3f920f54e

5️⃣ Criando o snapshot

aws ec2 create-snapshot --volume-id vol-051f64d8fb1d27d2d

Exemplo de retorno:
"SnapshotId": "snap-09430c398efc6daed"

Verificar se o snapshot foi criado:

aws ec2 wait snapshot-completed --snapshot-id snap-09430c398efc6daed

6️⃣ Reiniciando a instância

aws ec2 start-instances --instance-ids i-045ab79b3f920f54e

7️⃣ Automatizando snapshots via cron

Criação de snapshots a cada minuto:

echo "* * * * *  aws ec2 create-snapshot --volume-id vol-051f64d8fb1d27d2d 2>&1 >> /tmp/cronlog" > cronjob
crontab cronjob

Listar snapshots:

aws ec2 describe-snapshots --filters "Name=volume-id,Values=vol-051f64d8fb1d27d2d"

8️⃣ Limpando snapshots antigos
Remover cronjobs:

crontab -r
Visualizar script Python que mantém apenas os 2 mais recentes:

more /home/ec2-user/snapshotter_v2.py7

Executar limpeza:

python3.8 snapshotter_v2.py

9️⃣ Sincronizando arquivos com o bucket

Baixar e habilitar versionamento:

wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-3-124627/183-lab-JAWS-managing-storage/s3/files.zip

unzip files.zip

aws s3api put-bucket-versioning --bucket leandroglima --versioning-configuration Status=Enabled

Enviar arquivos:

aws s3 sync files s3://leandroglima/files/

Listar conteúdo:

aws s3 ls s3://leandroglima/files/

🔟 Testando controle de versão no S3

Deletar um arquivo:

rm files/file1.txt

aws s3 sync files s3://leandroglima/files/ --delete

Restaurar versão anterior:

aws s3api list-object-versions --bucket leandroglima --prefix files/file1.txt

aws s3api get-object --bucket leandroglima --key files/file1.txt --version-id <VERSION-ID> files/file1.txt

Confirmar restauração:

aws s3 sync files s3://leandroglima/files/

aws s3 ls s3://leandroglima/files/


## 💡 Resumo do que foi abordado:

- Criação e gerenciamento de buckets no S3

- Configuração de IAM Role para acesso seguro

- Criação manual e automatizada de snapshots EBS

- Limpeza automatizada de snapshots antigos

- Controle de versão e restauração de arquivos no S3

## 🗂 Visão Geral do Processo

<img width="1838" height="1035" alt="Imagem colada" src="https://github.com/user-attachments/assets/f9ebbe19-254c-46ad-bed3-78c56bee498d" />

<img width="1838" height="1035" alt="Imagem colada (2)" src="https://github.com/user-attachments/assets/4cf8b927-22f8-49b2-ae39-2c23f392996c" />

<img width="1838" height="1035" alt="Imagem colada (3)" src="https://github.com/user-attachments/assets/45839f1b-cb60-4672-9b53-0313f5580d40" />

<img width="1838" height="1035" alt="Imagem colada (4)" src="https://github.com/user-attachments/assets/edba7891-f4ac-446b-835a-7e1dc304795d" />

<img width="1838" height="697" alt="Imagem colada (14)" src="https://github.com/user-attachments/assets/adb9aeb6-2ebd-4afd-b6ea-b6820bb85ab4" />

<img width="1838" height="697" alt="Imagem colada (15)" src="https://github.com/user-attachments/assets/e6ee3058-f73f-4e1b-b660-f7a3f5e167dd" />

<img width="1838" height="697" alt="Imagem colada (17)" src="https://github.com/user-attachments/assets/a8e2fea6-f118-412b-85aa-5e541341a5dd" />

<img width="1838" height="697" alt="Imagem colada (16)" src="https://github.com/user-attachments/assets/9a0713f5-9b63-4b55-a256-c30fac1249c9" />

<img width="1838" height="824" alt="Imagem colada (5)" src="https://github.com/user-attachments/assets/7f632c79-86ec-4e83-ba58-2ce082cbe11d" />

<img width="1838" height="697" alt="Imagem colada (8)" src="https://github.com/user-attachments/assets/ddd3cb76-7eb4-401f-9d58-7e197ca496a4" />

<img width="1838" height="697" alt="Imagem colada (7)" src="https://github.com/user-attachments/assets/b6925485-31c0-48b5-b559-93ae42b1843b" />

<img width="1838" height="748" alt="Imagem colada (6)" src="https://github.com/user-attachments/assets/a37c7f36-ec73-4fa4-9f71-c9a15a3ef5ec" />

<img width="1838" height="697" alt="Imagem colada (11)" src="https://github.com/user-attachments/assets/fc997a3d-cb2a-4267-87fc-8925b680801d" />

<img width="1838" height="697" alt="Imagem colada (10)" src="https://github.com/user-attachments/assets/b5af6428-dbd6-446f-b659-cfd2a2bef738" />

<img width="1838" height="697" alt="Imagem colada (9)" src="https://github.com/user-attachments/assets/d5d4761d-46dd-4046-800c-9d28617864e9" />

<img width="1838" height="697" alt="Imagem colada (13)" src="https://github.com/user-attachments/assets/518540c3-d02e-4d4f-9330-7f810197cff3" />

<img width="1838" height="697" alt="Imagem colada (12)" src="https://github.com/user-attachments/assets/6a59a90f-05ab-449e-9f54-ffd6032d8913" />
