# Jellyfin ARR stack

Documento para guardar como foi feito a stack do Jellyfin/Arr.

## Pré requisitos
* Docker instalado
* Docker compose instalado
* Acesso root no instância
* ProtonVPN

## Criar pastas e dar as permissões

```bash
mkdir -p ./data/{torrents/{tv,movies,music},media/{tv,movies,music}}
apt install tree
tree ./data
chown -R 1000:1000 ./data
chmod -R a=,a+rX,u+w,g+w ./data
ls -ln ./data
```

## Configurar a VPN

### Pegar os valores do OpenVPN username dentro do painel da [ProtonVPN](https://account.protonvpn.com/account-password)

```bash
# Crie o arquivo .env
nano .env

# Adicione 
OPENVPN_USER={{PASTE_OPENVPN_USER}}
OPENVPN_PASSWORD={{PASTE_OPENVPN_PASSWORD}}
```

## Execute o docker compose:

```bash
docker compose up -d
```

# Configurações dos serviços

Siga os passos na ordem.

## qBittorrent: 

```bash
# Pegue a senha do admin que irá aparecer no log do container.
docker logs qbittorrent
```

* Abra: `http://<host_ip>:8085`
* Vá em `Tools -> Options -> WebUI` - Troque o usuário e a senha
* Crie as categorias - o path é o mesmo nome da categoria:
    * movies
    * tv
    * music

`* Crie as categorias primeiro senão a lista de categorias do menu irá desaparecer.`

* Vá em `Tools -> Options -> Downloads`
* Na parte do `Saving Management` valide essas configurações
    * `Default Torrent Management Mode - Automatic`
    * `When Torrent Category changed - Relocate torrent`
    * `When Default Save Path Changed - Switch affected torrents to Manual Mode`
    * `When Category Save Path Changed - Switch affected torrents to Manual Mode`
    * Marque os checkboxes `Use Subcategories` e `Use Category paths in Manual Mode`
    * Altere o Default save path para `/data/torrents`
    * Salve

<hr />

Para garantir que está usando a VPN, faça:
```bash
# Retorna o IP
docker exec -it qbittorrent curl ifconfig.me

# Valide no Whois IP
apt install whois
whois {{ IP }}

# Valide no https://ipleak.net/ com o link magnetico direto no qbittorrent
```

## Prowlarr:

* Abra `http://<host_ip>:9696`
* Crie o usuário
* Vá em `Settings -> Download Clients`
* Clique no `+` 
* Escolha o `qBittorrent`
* Configure o necessário
* Teste e Salve

## Radarr:

* Abra `http://<host_ip>:7878`
* Crie o usuário
* Vá em `Settings -> Media Management -> Add Root Folder`
* Adicione `/data/media/movies` como root folder
* Marque `click Show Advanced -> Importing -> Use Hardlinks instead of Copy` - garanta que esteja marcado
* Marque `Rename Movies`, `Delete empty movie folders during disk scan`
* Marque `Import Extra files` preencha com `srt,sub,nfo`
* Em `Settings -> Download clients` - adicione o `qBittorrent` - mesma coisa do Prowlarr
* Altere o `Category` para `movies`
* Teste e Salve
* Vá em `Settings -> General` - copie a `API key`

## Configure o Radarr no Prowlarr:

* Volte em `http://<host_ip>:9696`
* Vá em `Settings -> Apps` 
* Click no `+`
* Adicione o `Radarr`
* Cole a `API key` do Radarr
* Teste e Salve

## Sonarr:

* Abra `http://<host_ip>:8989`
* Crie o usuário
* Vá em `Settings -> Media Management -> Add Root Folder`
* Adicione `/data/media/tv` como root folder
* Marque `click Show Advanced -> Importing -> Use Hardlinks instead of Copy` - garanta que esteja marcado
* Marque `Rename Episodes`, `Delete empty Folders - delete empty series and season folders during disk scan`
* Marque `Import Extra files` preencha com `srt,sub,nfo`
* Em `Settings -> Download clients` - adicione o `qBittorrent` - mesma coisa do Prowlarr
* Altere o `Category` para `tv`
* Teste e Salve
* Vá em `Settings -> General` - copie a `API key`

## Configure o Sonarr no Prowlarr:

* Volte em `http://<host_ip>:9696`
* Vá em `Settings -> Apps` 
* Click no `+`
* Adicione o `Sonarr`
* Cole a `API key` do Sonarr
* Teste e Salve

## Lidarr:

* Abra `http://<host_ip>:8686`
* Crie o usuário
* Vá em `Settings -> Media Management -> +`
* Adicione `/data/media/music` como root folder
* Marque `click Show Advanced -> Importing -> Use Hardlinks instead of Copy` - garanta que esteja marcado
* Marque `Rename Tracks`, `Replace Illegal Characters`, `Delete empty folders`
* Em `Settings -> Download clients` - adicione o `qBittorrent` - mesma coisa do Prowlarr
* Altere o `Category` para `music`
* Teste e Salve
* Vá em `Settings -> General` - copie a `API key`

## Configure o Lidarr no Prowlarr:

* Volte em `http://<host_ip>:9696`
* Vá em `Settings -> Apps` 
* Click no `+`
* Adicione o `Lidarr`
* Cole a `API key` do Lidarr
* Teste e Salve

## Bazarr:

* Abra `http://<host_ip>:6767`
* Vá em `Settings -> Languages`, adicione o Pt-BR no `Languages Filter` e crie um `Language Profile` com o Pt-BR
* Vá em `Settings -> Providers` e adicione:
    * OpenSubtitles.org
    * Legendas.net
* Vá em `Settings -> Sonarr` e cole a `API KEY` do Sonarr
* Vá em `Settings -> Radarr` e cole a `API KEY` do Radarr
* Irá aparecer os menus do Sonarr e Radarr sincronizados.

## Configurar os indices no Prowlarr:

* Volte em `http://<host_ip>:9696`
* Vá em `Settings -> Indexers` 
* Click no `+`
* Adicione o `FlareSolverr`
* Em tags coloque `flare`
* Teste e Salve

Adicione o indexer para poder realizar as buscas.

* Vá em `Indexers`
* Clique em `Add Indexer`
* Procure por: `1337x`
* Clique e altere o `Seed Ratio` para `1`
* Preencha a `Tag` com `flare`
* Teste e Salve

## Restart geral

```bash
docker compose down
docker compose up -d
```

## Jellyfin

* Abra `http://<host ip address>:8096`
* Reralize a configuração inicial
* Configure os folders apontando para os diretorios de `/data/media`
    * `data/media/music`
    * `data/media/movies`
    * `data/media/tv`
* Modificar temas
    * [Awesome Jellyfin Themes](https://github.com/awesome-jellyfin/awesome-jellyfin/blob/main/THEMES.md)
    * Copie o `@import` do CSS
    * Vá em `Home -> User [Settings] -> Custom CSS Code`, cole o `@import`
    * Save

## Jellyseerr

* Abra `http://<host ip address>:5055`
* Reralize a configuração inicial
* Configure o Radarr e o Sonarr

<hr />

`Pirataria é crime`
