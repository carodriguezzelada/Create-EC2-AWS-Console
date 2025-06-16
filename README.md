<h1 id="gu%C3%ADacompleta%3Acrearinstanciasec2enlaconsolaaws">Guía Completa: Crear Instancias EC2 en la Consola AWS</h1>

<h2 id="%F0%9F%9A%80paso1%3Aaccederalaconsolaec2">🚀 Paso 1: Acceder a la Consola EC2</h2>

<ol>
<li><strong>Iniciar sesión</strong> en <a href="https://console.aws.amazon.com">AWS Console</a></li>
<li><strong>Buscar &#8220;EC2&#8221;</strong> en la barra de búsqueda superior</li>
<li><strong>Seleccionar</strong> el servicio EC2</li>
<li><strong>Verificar región</strong> en la esquina superior derecha (ej: us-east&#8211;1)</li>
</ol>

<hr />

<h2 id="%F0%9F%96%A5%EF%B8%8Fpaso2%3Acrearprimerainstanciaairviro">🖥️ Paso 2: Crear Primera Instancia (Airviro)</h2>

<h3 id="2.1iniciarelproceso">2.1 Iniciar el Proceso</h3>

<ol>
<li>Click en <strong>&#8220;Launch Instance&#8221;</strong> (botón naranja)</li>
<li><strong>Nombre</strong>: <code>Airviro-Production-Server</code></li>
</ol>

<h3 id="2.2seleccionaramisistemaoperativo">2.2 Seleccionar AMI (Sistema Operativo)</h3>

<ol>
<li>En <strong>&#8220;Application and OS Images&#8221;</strong>:</li>
<li>Click en <strong>&#8220;Browse more AMIs&#8221;</strong></li>
<li>En la pestaña <strong>&#8220;Community AMIs&#8221;</strong></li>
<li>Buscar: <code>Rocky Linux 9</code></li>
<li><strong>Seleccionar</strong>: Rocky Linux 9 (arquitectura x86_64)</li>
<li>✅ Verificar que sea <strong>&#8220;Free tier eligible&#8221;</strong> si aplica</li>
</ol>

<h3 id="2.3tipodeinstancia">2.3 Tipo de Instancia</h3>

<ol>
<li>En <strong>&#8220;Instance type&#8221;</strong>:</li>
<li><strong>Seleccionar</strong>: <code>c5.xlarge</code></li>
<li>📊 <strong>Especificaciones</strong>: 4 vCPUs, 8 GiB RAM</li>
<li>💰 <strong>Costo estimado</strong>: ~$0.17/hora</li>
</ol>

<h3 id="2.4keypairaccesossh">2.4 Key Pair (Acceso SSH)</h3>

<ol>
<li>En <strong>&#8220;Key pair (login)&#8221;</strong>:</li>
<li>Si tienes key pair: <strong>Seleccionar existente</strong></li>
<li>Si no tienes:

<ul>
<li>Click <strong>&#8220;Create new key pair&#8221;</strong></li>
<li><strong>Name</strong>: <code>airviro-keypair</code></li>
<li><strong>Key pair type</strong>: RSA</li>
<li><strong>Private key format</strong>: .pem</li>
<li>Click <strong>&#8220;Create key pair&#8221;</strong></li>
<li>💾 <strong>¡IMPORTANTE!</strong> Descargar y guardar el archivo .pem</li>
</ul></li>
</ol>

<h3 id="2.5configuraci%C3%B3ndered">2.5 Configuración de Red</h3>

<ol>
<li><p>En <strong>&#8220;Network settings&#8221;</strong> click <strong>&#8220;Edit&#8221;</strong>: </p></li>
<li><p><strong>VPC</strong>: Seleccionar tu VPC (o default) </p></li>
<li><p><strong>Subnet</strong>: Seleccionar subnet pública </p></li>
<li><p><strong>Auto-assign public IP</strong>: <strong>Enable</strong></p></li>
<li><p><strong>Security Group</strong> (Firewall): </p></li>
<li><p><strong>Create security group</strong>: ✅ Activado </p></li>
<li><p><strong>Security group name</strong>: <code>airviro-sg</code> </p></li>
<li><p><strong>Description</strong>: <code>Security group para servidor Airviro</code></p></li>
<li><p><strong>Reglas de seguridad</strong>:</p></li>
</ol>

<p><pre><code>Regla 1 (SSH):
- Type: SSH
- Protocol: TCP
- Port: 22
- Source: 0.0.0.0/0 (⚠️ Cambiar por tu IP para mayor seguridad)

Regla 2 (Aplicación Airviro):
- Type: Custom TCP
- Protocol: TCP
- Port: 8080
- Source: 0.0.0.0/0

Regla 3 (HTTPS - Opcional):
- Type: HTTPS
- Protocol: TCP
- Port: 443
- Source: 0.0.0.0/0</code></pre></p>

<h3 id="2.6configuraralmacenamiento">2.6 Configurar Almacenamiento</h3>

<ol>
<li>En <strong>&#8220;Configure storage&#8221;</strong>:</li>
<li><strong>Root volume</strong>:

<ul>
<li><strong>Size</strong>: <code>50 GiB</code></li>
<li><strong>Volume type</strong>: <code>gp3</code> (mejor rendimiento)</li>
<li><strong>IOPS</strong>: 3000 (default)</li>
<li><strong>Throughput</strong>: 125 MiB/s</li>
<li>✅ <strong>Encrypt</strong>: Activado</li>
<li>✅ <strong>Delete on termination</strong>: Activado</li>
</ul></li>
</ol>

<h3 id="2.7configuraci%C3%B3navanzada">2.7 Configuración Avanzada</h3>

<ol>
<li><p>En <strong>&#8220;Advanced details&#8221;</strong> (expandir): </p></li>
<li><p><strong>Monitoring</strong>: ✅ <strong>Enable detailed monitoring</strong> </p></li>
<li><p><strong>Termination protection</strong>: ✅ <strong>Enable</strong> (para producción)</p></li>
<li><p><strong>User data</strong> (script de inicialización):</p></li>
</ol>

<p><pre><code class="bash">#!/bin/bash
yum update -y
yum install -y htop wget curl git vim
hostnamectl set-hostname airviro-server

# Configurar timezone
timedatectl set-timezone America/Santiago

# Configurar firewall básico
systemctl enable firewalld
systemctl start firewalld
firewall-cmd --permanent --add-service=ssh
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

# Log de configuración
echo &quot;$(date): Servidor Airviro configurado exitosamente&quot; &gt;&gt; /var/log/setup.log</code></pre></p>

<h3 id="2.8tagsetiquetas">2.8 Tags (Etiquetas)</h3>

<ol>
<li>En <strong>&#8220;Tags&#8221;</strong> click <strong>&#8220;Add tag&#8221;</strong>:</li>
</ol>

<p><pre><code>Key: Name          Value: Airviro-Production
Key: Environment   Value: Production
Key: Application   Value: Airviro
Key: Owner         Value: [Tu nombre/equipo]
Key: CostCenter    Value: IT-AIRVIRO</code></pre></p>

<h3 id="2.9revisarylanzar">2.9 Revisar y Lanzar</h3>

<ol>
<li><strong>Revisar</strong> toda la configuración en el resumen</li>
<li>Click <strong>&#8220;Launch instance&#8221;</strong></li>
<li>✅ <strong>Confirmación</strong>: &#8220;Successfully initiated launch of instance&#8221;</li>
</ol>

<hr />

<h2 id="%F0%9F%8C%90paso3%3Acrearsegundainstancialaravel">🌐 Paso 3: Crear Segunda Instancia (Laravel)</h2>

<h3 id="3.1iniciarproceso">3.1 Iniciar Proceso</h3>

<ol>
<li>Click <strong>&#8220;Launch instance&#8221;</strong> nuevamente</li>
<li><strong>Name</strong>: <code>Laravel-Production-Server</code></li>
</ol>

<h3 id="3.2configuraci%C3%B3nlaravel">3.2 Configuración Laravel</h3>

<p><strong>Repetir pasos 2.2 (misma AMI Rocky Linux 9)</strong></p>

<p><strong>Tipo de instancia</strong>:<br/>
- <strong>Seleccionar</strong>: <code>t3.large</code><br/>
- 📊 <strong>Especificaciones</strong>: 2 vCPUs, 8 GiB RAM, Burst capability<br/>
- 💰 <strong>Costo</strong>: ~$0.083/hora</p>

<h3 id="3.3keypair">3.3 Key Pair</h3>

<ul>
<li><strong>Usar el mismo</strong> key pair creado anteriormente o crear uno nuevo</li>
</ul>

<h3 id="3.4networksettings-laravel">3.4 Network Settings - Laravel</h3>

<p><strong>Security Group específico</strong>:<br/>
- <strong>Security group name</strong>: <code>laravel-sg</code><br/>
- <strong>Description</strong>: <code>Security group para servidor Laravel</code></p>

<p><strong>Reglas</strong>:</p>
<pre><code>Regla 1 (SSH):
- Type: SSH, Port: 22, Source: 0.0.0.0/0

Regla 2 (HTTP):
- Type: HTTP, Port: 80, Source: 0.0.0.0/0

Regla 3 (HTTPS):
- Type: HTTPS, Port: 443, Source: 0.0.0.0/0

Regla 4 (Laravel Dev - Opcional):
- Type: Custom TCP, Port: 8000, Source: 0.0.0.0/0</code></pre>

<h3 id="3.5almacenamientolaravel">3.5 Almacenamiento Laravel</h3>

<ul>
<li><strong>Size</strong>: <code>30 GiB</code> (suficiente para Laravel)</li>
<li><strong>Volume type</strong>: <code>gp3</code></li>
<li>✅ <strong>Encrypt</strong>: Activado</li>
</ul>

<h3 id="3.6userdataparalaravel">3.6 User Data para Laravel</h3>
<pre><code class="bash">#!/bin/bash
yum update -y
yum install -y epel-release

# Instalar stack LAMP para Laravel
yum install -y httpd mariadb-server
yum install -y php php-cli php-fpm php-mysql php-json php-opcache php-mbstring php-xml php-gd php-curl php-zip

# Instalar Node.js (para assets)
curl -sL https://rpm.nodesource.com/setup_18.x | bash -
yum install -y nodejs

# Instalar Composer
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer

# Configurar servicios
systemctl enable httpd mariadb
systemctl start httpd mariadb

# Configurar hostname
hostnamectl set-hostname laravel-server

# Configurar timezone
timedatectl set-timezone America/Santiago

# Configurar firewall
systemctl enable firewalld
systemctl start firewalld
firewall-cmd --permanent --add-service=ssh
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload

# Crear directorio para Laravel
mkdir -p /var/www/html/laravel
chown apache:apache /var/www/html/laravel

echo &quot;$(date): Servidor Laravel configurado exitosamente&quot; &gt;&gt; /var/log/setup.log</code></pre>

<h3 id="3.7tagslaravel">3.7 Tags Laravel</h3>
<pre><code>Key: Name          Value: Laravel-Production
Key: Environment   Value: Production
Key: Application   Value: Laravel
Key: Owner         Value: [Tu nombre/equipo]
Key: CostCenter    Value: IT-LARAVEL</code></pre>

<hr />

<h2 id="%F0%9F%94%8Dpaso4%3Averificarinstancias">🔍 Paso 4: Verificar Instancias</h2>

<h3 id="4.1monitorearestado">4.1 Monitorear Estado</h3>

<ol>
<li>En <strong>EC2 Dashboard</strong> → <strong>&#8220;Instances&#8221;</strong></li>
<li><strong>Verificar estado</strong>:</li>
<li>✅ <strong>Instance State</strong>: Running</li>
<li>✅ <strong>Status Check</strong>: 2/2 checks passed</li>
</ol>

<h3 id="4.2obtenerinformaci%C3%B3ndeconexi%C3%B3n">4.2 Obtener Información de Conexión</h3>

<p><strong>Para cada instancia, anotar</strong>:<br/>
- <strong>Instance ID</strong>: i&#8211;0123456789abcdef0<br/>
- <strong>Public IPv4 address</strong>: 54.123.45.67<br/>
- <strong>Private IPv4 address</strong>: 10.0.1.100</p>

<hr />

<h2 id="%F0%9F%8C%90paso5%3Aconfiguraripsel%C3%A1sticasrecomendado">🌐 Paso 5: Configurar IPs Elásticas (Recomendado)</h2>

<h3 id="5.1crearelasticips">5.1 Crear Elastic IPs</h3>

<ol>
<li>En el menú EC2 → <strong>&#8220;Elastic IPs&#8221;</strong></li>
<li>Click <strong>&#8220;Allocate Elastic IP address&#8221;</strong></li>
<li><strong>Network Border Group</strong>: Mantener default</li>
<li><strong>Tags</strong>:</li>
<li>Name: <code>Airviro-EIP</code></li>
<li>Click <strong>&#8220;Allocate&#8221;</strong></li>
<li><strong>Repetir</strong> para Laravel con tag <code>Laravel-EIP</code></li>
</ol>

<h3 id="5.2asociaripsel%C3%A1sticas">5.2 Asociar IPs Elásticas</h3>

<ol>
<li><strong>Seleccionar</strong> la IP elástica</li>
<li>Click <strong>&#8220;Actions&#8221;</strong> → <strong>&#8220;Associate Elastic IP address&#8221;</strong></li>
<li><strong>Resource type</strong>: Instance</li>
<li><strong>Instance</strong>: Seleccionar instancia correspondiente</li>
<li>Click <strong>&#8220;Associate&#8221;</strong></li>
</ol>

<hr />

<h2 id="%F0%9F%94%90paso6%3Aconectarsealasinstancias">🔐 Paso 6: Conectarse a las Instancias</h2>

<h3 id="6.1usandosshlinuxmac">6.1 Usando SSH (Linux/Mac)</h3>
<pre><code class="bash"># Cambiar permisos del archivo key
chmod 400 ~/Downloads/airviro-keypair.pem

# Conectar a Airviro
ssh -i ~/Downloads/airviro-keypair.pem ec2-user@[IP_PUBLICA_AIRVIRO]

# Conectar a Laravel
ssh -i ~/Downloads/airviro-keypair.pem ec2-user@[IP_PUBLICA_LARAVEL]</code></pre>

<h3 id="6.2usandoputtywindows">6.2 Usando PuTTY (Windows)</h3>

<ol>
<li><strong>Convertir .pem a .ppk</strong> usando PuTTYgen</li>
<li><strong>Configurar PuTTY</strong>:</li>
<li>Host: IP pública de la instancia</li>
<li>Port: 22</li>
<li>Connection → SSH → Auth → Browse → Seleccionar archivo .ppk</li>
</ol>

<h3 id="6.3usarec2instanceconnectdesdeconsola">6.3 Usar EC2 Instance Connect (Desde consola)</h3>

<ol>
<li><strong>Seleccionar instancia</strong> en la consola</li>
<li>Click <strong>&#8220;Connect&#8221;</strong></li>
<li>Tab <strong>&#8220;EC2 Instance Connect&#8221;</strong></li>
<li><strong>User name</strong>: <code>ec2-user</code></li>
<li>Click <strong>&#8220;Connect&#8221;</strong></li>
</ol>

<hr />

<h2 id="%E2%9C%85paso7%3Averificaci%C3%B3npost-instalaci%C3%B3n">✅ Paso 7: Verificación Post-Instalación</h2>

<h3 id="7.1comandosdeverificaci%C3%B3n">7.1 Comandos de Verificación</h3>
<pre><code class="bash"># Verificar sistema
cat /etc/rocky-release
uname -a
df -h
free -h

# Verificar servicios (Laravel)
systemctl status httpd
systemctl status mariadb

# Verificar conectividad
curl -I http://localhost</code></pre>

<h3 id="7.2configuraci%C3%B3ninicialdeseguridad">7.2 Configuración Inicial de Seguridad</h3>
<pre><code class="bash"># Actualizar sistema
sudo yum update -y

# Configurar usuario adicional (opcional)
sudo adduser [tu-usuario]
sudo usermod -aG wheel [tu-usuario]

# Configurar fail2ban
sudo yum install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban</code></pre>

<hr />

<h2 id="%F0%9F%9A%A8checklistfinal">🚨 Checklist Final</h2>

<ul>
<li>[ ] ✅ Instancia Airviro (c5.xlarge) creada y funcionando</li>
<li>[ ] ✅ Instancia Laravel (t3.large) creada y funcionando</li>
<li>[ ] ✅ Security groups configurados correctamente</li>
<li>[ ] ✅ IPs elásticas asignadas</li>
<li>[ ] ✅ Conexión SSH funcional a ambas instancias</li>
<li>[ ] ✅ User data ejecutado correctamente</li>
<li>[ ] ✅ Tags aplicados para organización</li>
<li>[ ] ✅ Monitoreo habilitado</li>
<li>[ ] ✅ Encriptación de volúmenes activada</li>
</ul>

<hr />

<h2 id="%F0%9F%92%A1pr%C3%B3ximospasos">💡 Próximos Pasos</h2>

<ol>
<li><strong>Configurar aplicaciones específicas</strong> en cada servidor</li>
<li><strong>Implementar backup automatizado</strong></li>
<li><strong>Configurar monitoreo con CloudWatch</strong></li>
<li><strong>Establecer procedimientos de mantenimiento</strong></li>
<li><strong>Documentar arquitectura y procedimientos</strong></li>
</ol>

<hr />

<h2 id="%F0%9F%86%98troubleshootingcom%C3%BAn">🆘 Troubleshooting Común</h2>

<h3 id="problema%3Anopuedoconectarporssh">Problema: No puedo conectar por SSH</h3>

<p><strong>Solución</strong>:<br/>
- Verificar security group permite puerto 22<br/>
- Verificar IP pública asignada<br/>
- Verificar permisos del archivo .pem (chmod 400)</p>

<h3 id="problema%3Auserdatanoseejecut%C3%B3">Problema: User data no se ejecutó</h3>

<p><strong>Solución</strong>:<br/>
- Revisar logs: <code>sudo cat /var/log/cloud-init-output.log</code><br/>
- Verificar sintaxis del script</p>

<h3 id="problema%3Aaltocosto">Problema: Alto costo</h3>

<p><strong>Solución</strong>:<br/>
- Parar instancias cuando no se usen<br/>
- Considerar Reserved Instances para uso continuo<br/>
- Monitorear uso con AWS Cost Explorer</p>
