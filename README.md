Çoğu yerde bu sistem karışık halde anlatılıyor. Ben en basit şekilde anlatacağım. Ben de böyle kullanıyorum.

İlk önce discord dev tarafında url'nizi girmeniz ve ordan bir link almanız gerekmekte.

<img src="https://media.discordapp.net/attachments/967459506516811827/985590125440421929/unknown.png?width=924&height=450"/>

Örnek link: https://discord.com/api/oauth2/authorize?client_id=947199582813683822&redirect_uri=http%3A%2F%2Flocalhost%3A3000&response_type=code&scope=identify%20guilds

Bu linke girip izin veren biri girdiğiniz urlnin yanına code=<code> gelerek sitenize gider.

Örneğin: http://localhost:3000/?code=<code>

Biz bu code ile discord apisine bir istek atacağız.

Bu istekleri direkt olarak client tarafından değil, server tarafından atmanızı tavsiye ederim.

Server tarafında böyle dinliyoruz;

```js
app.post("/Token", (req, res) => {
    let { code } = req.body;
    if (!code) return res.status(400).send();
    let params = new URLSearchParams();
    params.append('client_id', "client id");
    params.append('client_secret', "client secret");
    params.append('grant_type', 'authorization_code');
    params.append('code', "<code> (urldeki)");
    params.append('redirect_uri', "http://localhost:3000");
    params.append('scope', 'identify', 'guilds');


    fetch("https://discord.com/api/oauth2/token", {
        method: "POST",
        body: params,
        headers: {
            "Content-type": "application/x-www-form-urlencoded"
        }
    }).then(a => a.json()).then(a => {
        res.status(200).send(a);
    });
});
```

Client tarafı ise;

```js
fetch("/Token", {
    method: "POST",
    headers: {
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        code: "<urldeki code>
    })
}).then(a => a.json()).then(a => {
    localStorage.setItem("RT", a.refresh_token);

    window.location.href = "/";
});
```

Evet artık refresh token elimizde, bu tokeni localStorage'a kaydettik. Discord apiyi kullanırken işimize yarayacak.

Discord apiyi kullandığımız zaman bu token değişiyor. O yüzden her seferinde yeni bir token almak gerek.

Bunu server tarafında şöyle dinleyelim;

```js
app.post("/RefreshToken", (req, res) => {
    let { rt } = req.body;
    if (!rt) return res.status(400).send();

    let params = new URLSearchParams();
    params.append('client_id', "client id");
    params.append('client_secret', "client secret");
    params.append('grant_type', 'refresh_token');
    params.append('refresh_token', `${rt}`);


    fetch("https://discord.com/api/oauth2/token", {
        method: "POST",
        body: params,
        headers: {
            "Content-type": "application/x-www-form-urlencoded"
        }
    }).then(a => a.json()).then(a => {
        res.status(200).send(a);
    });
});
```

Tamamdır.

Şimdi örnek bir tane veri çekelim discord apiden. Mesela me.

Server tarafından böyle dinlemeye alalım;

```js
app.post("/me", (req, res) => {
    let { a } = req.body;
    if (!a) return res.status(400).send();

    fetch("https://discord.com/api/users/@me", {
        headers: {
            authorization: `${a.token_type} ${a.access_token}`
        }
    }).then(a => a.json()).then(b => {
        res.status(200).send(b);
    });
});
```

Client tarafında ise şöyle bir istek yapacağız.

```js
fetch("/RefreshToken", {
    method: "POST",
    headers: {
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        rt: localStorage.getItem("RT")
    })
}).then(a => a.json()).then(async a => {
    localStorage.setItem("RT", a.refresh_token);
    
    await fetch("/me", {
        method: "POST",
        headers: {
            'Accept': 'application/json, text/plain, */*',
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            a
        })
    }).then(a => {
        return a.json();
    }).then(user => {
        console.log(user) // User verisi
    });
});
```
