# Configuração de Rede no Virtual Box

Para que possamos executar os laboratórios no laboratório da FIAP, é importante realizarmos uma pequena alteração nas configurações de rede das máquinas virtuais, uma vez que precisamos fazer com que as duas máquinas virtuais utilizadas nos próximos passos conversem entre si e sejam acessíveis a partir de nosso computador, vamos realizar os seguintes passos.

## 1. Criar uma rede NAT

Na interface do VirtualBox, clique em `Arquivo` e em seguida em `Preferências`. Feito isto, clique na opção `Rede` no menu lateral esquerdo:

![network settings](/02-ConfiguracaoRedeVirtualBox/images/network_settings.png)

Feito isto, clique no botão com o sinal de `+` no canto direito desta tela, para criarmos uma nova rede NAT. Em seguida, clique em OK:

![new nat network](/02-ConfiguracaoRedeVirtualBox/images/new_nat_network.png)

## 2. Configurando as máquinas virtuais

Após criar a rede NAT, selecione a máquina virtual `automation-server` na interface inicial do VirtualBox e clique em `Configurações`.

![server settings](/02-ConfiguracaoRedeVirtualBox/images/server_settings.png)

Clique então em `Rede` e selecione a opção `Rede NAT`. Ao finalizar esta configuração, clique em OK:

![select nat network](/02-ConfiguracaoRedeVirtualBox/images/select_nat_network.png)

Repita o mesmo processo para a máquina virtual `automation-client`

## 3. Obtendo os endereços IP

Quando configurarmos a interface de rede para utilizar a opção `Rede NAT` e iniciarmos as máquinas virtuais, o VirtualBox irá entregar um endereço IP para cada uma das máquinas virtuais utilizando um DHCP Server próprio. Devido a isto, precisaremos obter o endereço IP entregue a cada uma das máquinas virtuais para que possamos então configurar o encaminhamento de portas.

Para obter o endereço IP, inicie o `automation-server` e através da interface do VirtualBox acesse o mesmo utilizando as credenciais:

    Usuário: automation-admin
    Senha: automationserver

Após efetuar login no sistema operacional, execute o comando:

    $ ifconfig enp0s3

A saída do comando deverá ser semelhante a:

    enp0s3    Link encap:Ethernet  HWaddr 08:00:27:b0:90:09  
              inet addr:10.0.2.4  Bcast:10.0.2.255  Mask:255.255.255.0
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:341 errors:0 dropped:0 overruns:0 frame:0
              TX packets:294 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000
              RX bytes:287987 (287.9 KB)  TX bytes:32422 (32.4 KB)

Note que o endereço IP do servidor é exibido logo na segunda linha da saída do comando logo após `ined addr:`. Em nosso exemplo, o endereço IP é `10.0.2.4`.

> Note que o endereço IP de sua máquina virtual pode ser diferente do endereço apresentado no exemplo anterior.

Tome nota do endereço IP do `automation-server` e realize o mesmo procedimento com a máquina virtual `automation-client` para obter o endereço IP da mesma utilizando as credenciais:

    Usuário: automation-admin
    Senha: automationclient


## 4. Configurando o Encaminhamento de Portas

Após obter os endereços IP atribuídos via DHCP a ambas as máquinas virtuais, vamos realizar a configuração do encaminhamento de portas, para que possamos acessar as VM's utilizando o putty.

Para isto, na console do VirtualBox clique em `Arquivo`, `Preferências` e selecione a opção `Rede` no menu lateral esquerdo. Nesta tela, a nossa rede NAT será exibida. Clique então no terceiro botão na lateral direita para editar as configurações:

![nat configurations](/02-ConfiguracaoRedeVirtualBox/images/nat_configurations.png)

Em seguida, clique em `Encaminhamento de portas`:

![port forwarding](/02-ConfiguracaoRedeVirtualBox/images/port_forwarding.png)

Vamos então clicar no sinal de `+` no lado direito da tela e adicionar os mapeamentos. Deveremos criar os seguintes mapeamentos de porta:

| Porta do Host | IP do Convidado   | Porta do Convidado |
|---------------|-------------------|--------------------|
| 22            | IP do automation-server | 22                 |
| 80            | IP do automation-server | 80                 |
| 443           | IP do automation-server | 443                |
| 23            | IP do automation-client | 22                 |

Conforme a imagem abaixo:

![port forwarding rules](/02-ConfiguracaoRedeVirtualBox/images/port_forwarding_rules.png)

Feito isto, salve todas as configurações e utilize o putty para realizar acesso remoto, lembrando que para acesso ao `automation-server` vamos utilizar a porta `22` e para o `automation-client` vamos utilizar a porta `23`:

![putty access](/02-ConfiguracaoRedeVirtualBox/images/putty_access.png)

## 5. Reconfigurando o automation-server

Caso o endereço IP provido ao seu `automation-server` seja diferente do endereço utilizado anteriormente para as configurações, será necessário refazermos o processo de configuração para geração de certificados. Caso este seja o cenário, no `automation-server` execute os seguintes comandos:

    echo api_fqdn "\""$(ifconfig enp0s3 | grep 'inet addr' | cut -d: -f2 | awk '{print $1}')"\"" | \
    sudo tee --append /etc/opscode/automation-server.rb > /dev/null

E em seguida:

    $ sudo automation-server-ctl reconfigure
    $ sudo chef-manage-ctl reconfigure

Feito isto, realize novamente o download do `Starter Kit` e disponibilize o mesmo em seu workspace. Ao extrair o `Starter Kit`, acesse o diretório `chef-repo` e execute o comando:

    $ knife ssl fetch

Valide o funcionamento através do comando:

    $ knife client list

## 6. Removendo o node anterior

Em nossa ultima aula, utilizamos o knife para realizar o bootstrap de um container, simulando o servidor `motd-server`. Como tratava-se de um container, o mesmo não existe mais quando reiniciamos o servidor, e o processo de bootstrap deverá ser realizado novamente. Neste caso, acesse a interface web do `automation-server` e remova o node criado anteriormente selecionando o mesmo e clicando em `Delete` no menu lateral esquerdo:

![delete node](/02-ConfiguracaoRedeVirtualBox/images/delete_node.png)

Feito isto, inicie um novo container e siga os passos para realizar o bootstrap novamente de acordo com o tutorial [05-BootstrapUsandoKnife](/05-BootstrapUsandoKnife/).
