# Galaxy の外部ユーザ認証機能を利用する

## Galaxy 設定

galaxy.ini:
```
use_remote_user = True
```

## Nginx 設定

Nginx を Reverse Proxy として使用し、Basic 認証する方式を検証してみる。

/etc/nginx/nginx.conf:
```
    server {
        location /galaxy/ {
            proxy_pass http://GALAXY_HOST/;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            auth_basic "Please enter username and password.";
            auth_basic_user_file "/etc/nginx/htpasswd";
        }
    }
```

/etc/nginx/htpasswd:
```
test01@example.com:*****
```

## Galaxy の外部ユーザ認証機能について

- Nginxによる Reverse Proxy が `REMOTE_USER` の HTTP ヘッダを付けて
  Galaxy にリクエストすると、Galaxy は認証ユーザとしてそれを認識する仕組みが備わっている。

- Proxy から受け取る HTTP ヘッダは変更可能
  * デフォルト設定では `REMOTE_USER` が使われるが、 `remote_user_header` で変更可能

- その他の設定
  * https://github.com/galaxyproject/galaxy/blob/master/config/galaxy.ini.sample
  * 最新版 (release 18.01-) では `galaxy.yml` の YAML 形式になったらしい。


## Galaxy ログイン時の挙動

- Galaxy に登録済みユーザの場合
  * 外部認証 OK の場合、Galaxyでログイン済みの状態となる。

- Galaxy に存在しないユーザの場合
  * 外部認証 OK の場合、Galaxyで登録＆ログイン済みの状態となる。
  * `galaxy_user` テーブルに `external = 't'` で登録される。
    > ローカル認証に戻す場合は `external = 'f'` に変更する必要がある。

- Galaxy に登録済みユーザだが、外部認証とGalaxy側のパスワードが異なる場合
  * 外部認証 OK の場合、Galaxyでログイン済みの状態となる。
  * Galaxy側で登録時のパスワードは使われない。


## TODO

- 認証Proxy を SAML ベースの SSO で認証する外部サービスと連携可能な SP として構築して使えるか？


## Reference

https://galaxyproject.org/admin/config/nginx-external-user-auth/  

https://galaxyproject.org/admin/config/external-user-auth/

