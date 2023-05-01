# Unbound: Servidor DNS Recursivo

## O que é o Unbound?

O Unbound é um servidor DNS recursivo que tem a capacidade de pesquisar informações de DNS em nome de seus clientes, permitindo que nomes de domínio sejam traduzidos em endereços IP. Projetado para ser rápido, seguro e escalável, o Unbound é uma solução de software livre que é frequentemente usado para melhorar a privacidade e segurança online. Além disso, ele possui uma arquitetura modular e pode ser configurado para suportar várias extensões, tornando-o altamente personalizável e flexível.

Algumas das principais características e funcionalidades do Unbound incluem:

Segurança: O Unbound é projetado para ser resistente a ataques de DNS spoofing e cache poisoning. Ele utiliza criptografia DNSSEC para garantir que as respostas DNS recebidas sejam autênticas e não foram modificadas durante a transmissão.

Velocidade: O Unbound é otimizado para alto desempenho e baixa latência. Ele usa técnicas de caching e pre-fetching para minimizar o tempo de resposta e reduzir a sobrecarga do servidor.

Escalabilidade: O Unbound é capaz de lidar com grandes volumes de consultas DNS simultâneas e pode ser facilmente escalado adicionando mais servidores.

Flexibilidade: O Unbound é altamente configurável e pode ser personalizado para atender às necessidades específicas de um ambiente de rede. Ele suporta vários tipos de consultas DNS, incluindo consultas recursivas e consultas diretas.

Fácil integração: O Unbound é compatível com vários sistemas operacionais, como Linux, FreeBSD, macOS e Windows. Ele também pode ser integrado com outros servidores DNS, como BIND e PowerDNS.

&nbsp;

## Como o Unbound funciona:

1. Recebimento de consultas: O Unbound recebe consultas DNS de seus clientes (usuários finais ou outros servidores DNS). Essas consultas incluem um nome de domínio que precisa ser resolvido em um endereço IP.

2. Resolução recursiva: O Unbound inicia o processo de resolução recursiva, onde ele envia a consulta DNS para o servidor raiz (root server) responsável pelo domínio de nível superior (TLD) correspondente ao nome de domínio. O servidor raiz responde com uma lista de servidores DNS autoritativos para o TLD correspondente.

3. Consulta aos servidores autoritativos: O Unbound envia a consulta DNS para um dos servidores DNS autoritativos listados pelo servidor raiz. O servidor autoritativo responde com um registro de recurso (RR) que contém informações sobre o nome de domínio, como um registro A (endereço IP) ou um registro MX (servidor de e-mail).

4. Caching: O Unbound armazena em cache a resposta DNS recebida do servidor autoritativo para acelerar consultas futuras para o mesmo nome de domínio. Isso ajuda a reduzir o tempo de resposta e a carga no servidor.

5. Retorno da resposta: O Unbound retorna a resposta DNS ao cliente que iniciou a consulta original. O cliente agora tem o endereço IP correspondente ao nome de domínio que estava tentando resolver.

&nbsp;

![Diagrama de funcionamento do Unbound](/imagens/diagrama-unbound.png)

&nbsp;

No diagrama acima, o cliente faz uma consulta DNS para o servidor Unbound. O servidor Unbound, em seguida, inicia o processo de resolução recursiva, enviando a consulta DNS para o servidor raiz responsável pelo TLD correspondente. O servidor raiz responde com uma lista de servidores DNS autoritativos para o TLD correspondente, que o servidor Unbound usa para consultar um servidor autoritativo específico. O servidor autoritativo responde com a resposta DNS contendo as informações solicitadas, que o servidor Unbound, em seguida, retorna ao cliente.

É importante destacar que durante todo esse processo de resolução recursiva, o servidor Unbound utiliza seu cache local para armazenar as respostas de consultas anteriores. Isso significa que se o servidor Unbound já tiver a resposta em seu cache local, ele não precisará passar por todo o processo de consulta recursiva, o que torna o processo de resolução de consultas DNS muito mais rápido.

&nbsp;

## DNSSEC (Domain Name System Security Extensions)

O DNSSEC é uma extensão de segurança do DNS que foi projetada para proteger o DNS contra ataques de falsificação de DNS. O DNSSEC utiliza criptografia de chave pública para assinar digitalmente os registros DNS, permitindo que os clientes possam verificar se os registros DNS recebidos são autênticos e não foram adulterados.

O Unbound pode verificar a autenticidade dos registros DNS usando DNSSEC. Quando um cliente faz uma consulta DNS, o Unbound envia a consulta para o servidor raiz responsável pelo TLD correspondente, que responde com uma lista de servidores DNS autoritativos para o TLD e chaves de assinatura DNSSEC. O Unbound usa as chaves de assinatura para verificar a autenticidade dos registros DNS recebidos dos servidores autoritativos.

Se o registro DNS não for autêntico ou não puder ser verificado, o Unbound não irá retornar o registro para o cliente, garantindo a integridade e autenticidade das informações DNS. Isso ajuda a proteger os usuários contra ataques de falsificação de DNS e aumenta a segurança e confiabilidade do sistema de resolução de nomes de domínio.

&nbsp;

## Instalação do Unbound

O Unbound pode ser configurado de três maneiras:

1. Como um DNS de cache recursivo simples, usando a porta UDP 53 sem criptografia.

2. Como um DNS sobre TLS, com cache recursivo DNS, usando a porta TCP 853 ENCRYPTED.

3. Como um DNS autoritativo, validador e de cache recursivo.

Para maior segurança, neste exemplo, utilizaremos a terceira configuração. O Unbound consultará recursivamente qualquer nome de host dos servidores DNS raiz dos quais não possui uma cópia em cache e validará as consultas usando DNSSEC e bits aleatórios codificados em 0x20 para evitar tentativas de falsificação. Além disso, esse servidor podera ser o DNS autorizado para alguns nomes de host em sua LAN.

Para iniciar, algumas etapas de configuração são necessárias. Primeiramente, é preciso obter uma cópia atualizada da lista de servidores DNS raiz, denominada root.hints. Em seguida, é necessário obter a configuração da chave confiável raiz DNSSEC. Por fim, é preciso configurar quaisquer nomes de host e endereços IP de sua LAN. 

&nbsp;

### Instalando o Unbound

O Unbound está disponível nos repositórios oficiais do Ubuntu. Para instalar o Unbound, execute o seguinte comando:

```bash
sudo apt install unbound
```

&nbsp;

### Configurando o Unbound

O arquivo de configuração padrão do Unbound é `/etc/unbound/unbound.conf`. Caso queira aprender mais sobre todas as opções de configuração disponíveis, o arquivo de configuração padrão vem com muitos comentários que explicam o que cada opção faz.

Para este exemplo, vamos utilizar o arquivo de configuração [unbound.conf](unbound.conf) que ficará no diretório `/etc/unbound/unbound.conf.d/`

```bash
sudo nano /etc/unbound/unbound.conf.d/unbound.conf
```

&nbsp;

### Obtendo a lista de servidores DNS raiz

A lista de servidores DNS raiz é um arquivo chamado `root.hints` que contém os endereços IP dos servidores raiz. O Unbound usa esse arquivo para iniciar o processo de resolução recursiva. 

O arquivo root.hints pode ser obtido no site do IANA (Internet Assigned Numbers Authority) em https://www.iana.org/domains/root/servers.

```bash
wget https://www.internic.net/domain/named.root -O /var/lib/unbound/root.hints
```

&nbsp;

### Obtendo a configuração da chave confiável raiz DNSSEC

O Unbound usa a configuração da chave confiável raiz DNSSEC para verificar a autenticidade dos registros DNS recebidos dos servidores DNS raiz. A configuração da chave confiável raiz DNSSEC pode ser obtida no site do IANA em https://www.iana.org/dnssec/files mas essa chave não é atualizada com frequência então você pode usar os valores abaixo ou verificar se existe uma versão mais atualizada em https://data.iana.org/root-anchors/root-anchors.xml.

```bash
IN DS 19036 8 2 49AAC11D7B6F6446702E54A1607371607A1A41855200FD2CE1CDDE32F24E8FB5
IN DS 20326 8 2 E06D44B80B8F1D39A95C0B0D7C65D08458E880409BBC683457104237C7F8EC8D
```

Crie o arquivo root.key e adicione os valores acima.

```bash
sudo nano /etc/unbound/root.key
```

Certifique-se de que o arquivo root.key é de propriedade do usuário com o qual o daemon do Unbound está sendo executado, neste exemplo estamos utilizando o usuário `unbound`.

```bash
sudo chown unbound:unbound /etc/unbound/root.key
```

Habilite e reinicie o serviço do Unbound.

```bash
sudo systemctl enable unbound
sudo systemctl restart unbound
```

&nbsp;

## Zonas atendidas localmente

O Unbound pode ser configurado para atender localmente a determinadas zonas. Isso é útil para nomes de host que não são resolvidos corretamente pelos servidores DNS raiz. Por exemplo, se você tiver um servidor de arquivos chamado files.example.com, poderá configurar o Unbound para atender localmente a esse nome de host. Isso significa que o Unbound não precisará consultar os servidores DNS raiz para resolver esse nome de host.

No exemplo, é configurado o nome de host fileserver.home.lan resolvendo para o endereço IP 10.2.2.1. Além disso, é configurada uma pesquisa reversa para permitir que 10.2.2.1 resolva de volta para o nome do host fileserver.home.lan.

Isso pode ser feito adicionando as seguintes linhas no arquivo /etc/unbound/unbound.conf.d/unbound.conf:

```bash

# locally served zones can be configured for the machines on the LAN.

    local-zone: "home.lan." static

    local-data: "fileserver.home.lan.  IN A 10.2.2.1"

    local-data-ptr: "10.2.2.1  fileserver.home.lan"

```

&nbsp;

## Testando o Unbound

Para testar se o Unbound está funcionando corretamente e com o DNSSEC habilitado, execute o seguinte comando:

```bash
dig com. SOA +dnssec
```

Se o Unbound estiver funcionando corretamente, você verá a flag `ad` no resultado:

&nbsp;

## Resolvendo problemas

### Resolvendo nomes de host localmente

Para que o servidor que está rodando o Unbound possa resolver nomes de host, pode ser necessário configurar o arquivo `/etc/resolv.conf` para que o Unbound seja o servidor DNS padrão.

```bash
sudo nano /etc/resolv.conf
```

Adicione a linha abaixo no arquivo `/etc/resolv.conf` para que o Unbound seja o servidor DNS padrão.

```bash
nameserver 127.0.0.1
```

Após isso desative o serviço `systemd-resolved` para que o arquivo `/etc/resolv.conf` não seja sobrescrito.

```bash
sudo systemctl disable systemd-resolved
```

Reinicie o sistema para que as alterações tenham efeito.

```bash
sudo reboot
```

&nbsp;

### Resolvendo nomes de host em outra rede ou sub-rede

Para que os dispositivos de outra rede ou sub-rede possam resolver nomes de host, pode ser necessário adicionar a interface de rede no arquivo `/etc/unbound/unbound.conf.d/unbound.conf`.

```bash
sudo nano /etc/unbound/unbound.conf.d/unbound.conf
```

Adicione a linha abaixo no arquivo `/etc/unbound/unbound.conf.d/unbound.conf` para que o Unbound possa resolver nomes de host em outra rede ou sub-rede.

```bash
interface: xxx.xxx.xxx.xxx
```

Além disso, pode ser necessário adicionar o endereço IP de rede ou do dispositivo no arquivo `/etc/unbound/unbound.conf.d/unbound.conf`. Para isso adicione a linha abaixo no arquivo `/etc/unbound/unbound.conf.d/unbound.conf` para que o Unbound possa resolver nomes de host em outra rede ou sub-rede.

```bash
access-control: xxx.xxx.xxx.xxx/xx allow
```

Reinicie o sistema para que as alterações tenham efeito.

```bash
sudo reboot
```
