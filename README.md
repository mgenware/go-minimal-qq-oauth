# go-minimal-qq-oauth
A minimal library for QQ OAuth. Tested on Go 1.6.

## Installation
```sh
go get github.com/mgenware/go-minimal-qq-oauth
```

# Example
```go
package main

import (
	"fmt"
	"net/http"

	"github.com/mgenware/go-minimal-qq-oauth"
)

const (
	clientID     = "{{Your Client ID}}"
	clientSecret = "{{Your Client Secret}}"
	redirectionURL  = "{{Your Redirection URL}}"
	urlState     = "{{Some random string}}"
)

var qq *qqOAuth.OAuth

func init() {
	var err error
	qq, err = qqOAuth.NewQQOAuth(clientID, clientSecret, redirectionURL)
	if err != nil {
		panic(err)
	}

	qqOAuth.Logging = true
}

func oauthHandler(w http.ResponseWriter, r *http.Request) {
	urlStr, err := qq.GetAuthorizationURL(urlState)
	if err != nil {
		w.Write([]byte(err.Error()))
		return
	}
	http.Redirect(w, r, urlStr, http.StatusMovedPermanently)
}

func callbackHandler(w http.ResponseWriter, r *http.Request) {
	code := r.FormValue("code")
	if code == "" {
		w.Write([]byte("Invalid code"))
		return
	}

	state := r.FormValue("state")
	if state != urlState {
		w.Write([]byte("Invalid state"))
		return
	}

	token, err := qq.GetAccessToken(code)
	if err != nil {
		w.Write([]byte(err.Error()))
		return
	}

	openid, err := qq.GetOpenID(token.AccessToken)
	if err != nil {
		w.Write([]byte(err.Error()))
		return
	}

	profile, err := qq.GetUserInfo(token.AccessToken, openid.OpenID)
	if err != nil {
		w.Write([]byte(err.Error()))
		return
	}
	w.Write([]byte(fmt.Sprint(profile)))
}

func main() {
	http.HandleFunc("/qq_oauth", oauthHandler)
	http.HandleFunc("/qq_oauth_callback", callbackHandler)
	http.ListenAndServe(":3000", nil)
}

```
