
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


