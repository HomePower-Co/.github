# Guía de Configuración de AWS CLI con SSO y Conexión SSM

Esta guía detalla el proceso completo para configurar AWS CLI, autenticarse mediante AWS SSO, instalar el plugin de Session Manager y conectarse a un bastion host utilizando SSM.

## Tabla de Contenidos

- [Instalación de AWS CLI](#instalación-de-aws-cli)
- [Configuración de AWS SSO](#configuración-de-aws-sso)
- [Instalación del Plugin de Session Manager](#instalación-del-plugin-de-session-manager)
- [Conexión a Bastion Host con SSM](#conexión-a-bastion-host-con-ssm)
- [Solución de Problemas](#solución-de-problemas)

---

## Instalación de AWS CLI

### Ubuntu/Linux

#### Opción 1: Instalación mediante script oficial (Recomendado)

```bash
# Descargar el instalador
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Descomprimir el archivo
unzip awscliv2.zip

# Ejecutar el instalador
sudo ./aws/install

# Verificar la instalación
aws --version
```

### Windows

1. Descargar el instalador MSI desde: https://awscli.amazonaws.com/AWSCLIV2.msi
2. Ejecutar el instalador
3. Abrir PowerShell o CMD y verificar:

```powershell
aws --version
```

---

## Configuración de AWS SSO

### Paso 1: Configurar el perfil de SSO

```bash
aws configure sso
```

Durante la configuración, proporciona la siguiente información:

```
SSO session name (Recommended): hpowerco
SSO start URL [None]: https://hpowerco.awsapps.com/start/#
SSO region [None]: us-east-2
SSO registration scopes [sso:account:access]:
```

### Paso 2: Autenticarse

El comando anterior abrirá automáticamente tu navegador para completar la autenticación. Si no se abre automáticamente:

1. Copia la URL proporcionada en la terminal
2. Pégala en tu navegador
3. Ingresa el código de verificación
4. Completa la autenticación

### Paso 3: Seleccionar cuenta y rol

```
There are N AWS accounts available to you.
> developmente (123456789012)

Using the account ID 123456789012
There are M roles available to you.
> AdministratorAccess
CLI default client Region [None]: us-east-2
CLI default output format [None]: json
CLI profile name [AdministratorAccess-123456789012]: hpowerco-dev
```

### Paso 5: Iniciar sesión con SSO

```bash
# Primera vez o cuando la sesión expire
aws sso login --profile hpowerco-dev
```

### Configurar perfil predeterminado (linux)

```bash
# Establecer variable de entorno linux
export AWS_PROFILE=hpowerco-dev
```

### Configurar perfil predeterminado (windows)

```powershell
# Establecer variable de entorno linux
SET AWS_PROFILE=hpowerco-dev
```
---

## Instalación del Plugin de Session Manager

El plugin de Session Manager permite utilizar AWS Systems Manager Session Manager para conexiones SSH y tunelización de puertos.

### Ubuntu/Linux

```bash
# Descargar el plugin
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/ubuntu_64bit/session-manager-plugin.deb" -o "session-manager-plugin.deb"

# Instalar
sudo dpkg -i session-manager-plugin.deb

# Verificar la instalación
session-manager-plugin --version
```

### Windows

1. Descargar el instalador desde: https://s3.amazonaws.com/session-manager-downloads/plugin/latest/windows/SessionManagerPluginSetup.exe
2. Ejecutar el instalador
3. Verificar en PowerShell:

```powershell
session-manager-plugin
```

---


### Conexión a database mediante Session Manager (linux)

```bash
# Túnel a puerto local
aws ssm start-session \
  --target $(aws ec2 describe-instances --filters "Name=tag:Name,Values=bastion" "Name=instance-state-name,Values=running" --query "Reservations[*].Instances[*].InstanceId" --output text) \
  --document-name AWS-StartPortForwardingSessionToRemoteHost \
  --parameters '{"host":["host-to-conect"],"portNumber":["5432"],"localPortNumber":["5434"]}'
```
### Conexión a database mediante Session Manager (Windows)

#### Declaracion del archivo de parametro
Este archivo debe contener los parámetros y valor necesarios para poder realizar una conexión mediante Session Manager, de la siguiente manera debe verse el archivo de parámetros:

```bash
{
    "host": [
       "host-to-conect"
    ],
    "portNumber": [
        "5432"
    ],
    "localPortNumber": [
        "5432"
    ]
}
```
La conexión se realiza de la siguiente manera:

```bash
# Túnel a puerto local
aws ssm start-session --target  i-0f7e8428b6bb27690  --document-name AWS-StartPortForwardingSessionToRemoteHost --parameters file://params.json --region us-east-2
```



### Obtener datos de conexión

- Iniciar sección en https://hpowerco.awsapps.com/start/#
- Ir al servicio de `secrets manager`
- Seleccionar el servicio al cual quiere obtener los datos
- Resolver los secretos

---

## Referencias

- [Documentación oficial de AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/)
- [Configuración de AWS SSO](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)
- [Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)
- [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)

---
