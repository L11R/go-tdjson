# ⚠️ DEPRECATION NOTICE
Please, consider this small library as early proof of concept. I wrote it right after TDLib was released. There are more mature libraries currently, I personally can highly recommend [this one](https://github.com/zelenin/go-tdlib) from [@zelenin](https://github.com/zelenin). I use it in my private projects and can definitely say that it's battle-ready.

# Golang bindings for TDLib (Telegram MTProto library)
[![GoDoc](https://godoc.org/github.com/L11R/go-tdjson?status.svg)](https://godoc.org/github.com/L11R/go-tdjson)

In addition to the built-in methods:
- Destroy()
- Execute()
- Receive()
- Send()
- SetFilePath()
- SetLogVerbosityLevel()

It also has two interesting methods:
- Auth()
- SendAndCatch()

# Linking statically against TDLib
I recommend you to link it statically if you don't want compile TDLib on production (don't forget that it requires at least 8GB of RAM). 
<br/>To do that, just build your source with tag `tdjson_static`: `go build -tags tdjson_static`
<br />For more details read this issue: https://github.com/tdlib/td/issues/8

# Example
```golang
package main

import (
	"github.com/L11R/go-tdjson"
	"fmt"
	"log"
	"os"
	"os/signal"
	"syscall"
)

func main() {
	tdjson.SetLogVerbosityLevel(1)
	tdjson.SetFilePath("./errors.txt")

	var params []tdjson.Option = []tdjson.Option{
		tdjson.WithMessageDatabase(),
		tdjson.WithStorageOptimizer(),
	}

	// Get API_ID and API_HASH from env vars
	apiId := os.Getenv("API_ID")
	if apiId == "" {
		log.Fatal("API_ID env variable not specified")
	}
	params = append(params, tdjson.WithID(apiId))

	apiHash := os.Getenv("API_HASH")
	if apiHash == "" {
		log.Fatal("API_HASH env variable not specified")
	}
	params = append(params, tdjson.WithHash(apiHash))

	// Create new instance of client
	client := tdjson.NewClient(params...)

	// Handle Ctrl+C
	ch := make(chan os.Signal, 2)
	signal.Notify(ch, os.Interrupt, syscall.SIGTERM)
	go func() {
		<-ch
		client.Destroy()
		os.Exit(1)
	}()

	// Main loop
	for update := range client.Updates {
		// Show all updates
		fmt.Println(update)

		// Authorization block
		if update["@type"].(string) == "updateAuthorizationState" {
			if authorizationState, ok := update["authorization_state"].(tdjson.Update)["@type"].(string); ok {
				res, err := client.Auth(authorizationState)
				if err != nil {
					log.Println(err)
				}
				log.Println(res)
			}
		}
	}
}
```
