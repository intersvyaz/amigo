# amigo
Asterisk AMI connector on golang.

Attention!
API has been changed in v0.1.2. Please read the godoc.

Usage is pretty simple.

## Installation:

`go get github.com/intersvyaz/amigo`

## Using
Import module to your project:
```go
import "github.com/intersvyaz/amigo"
```

Then use:
```go
package main

import (
	"fmt"

	"github.com/intersvyaz/amigo"
)

// Creating hanlder functions
func DeviceStateChangeHandler(m map[string]string) {
	fmt.Printf("DeviceStateChange event received: %v\n", m)
}

func DefaultHandler(m map[string]string) {
	fmt.Printf("Event received: %v\n", m)
}

func main() {
	fmt.Println("Init Amigo")

	settings := &amigo.Settings{Username: "username", Password: "password", Host: "host"}
	a := amigo.New(settings)

	a.Connect()

	// Listen for connection events
	a.On("connect", func(message string) {
		fmt.Println("Connected", message)
	})
	a.On("error", func(message string) {
		fmt.Println("Connection error:", message)
	})

	// Registering handler function for event "DeviceStateChange"
	a.RegisterHandler("DeviceStateChange", DeviceStateChangeHandler)

	// Registering default handler function for all events.
	a.RegisterDefaultHandler(DefaultHandler)

	// Optionally create channel to receiving all events
	// and set created channel to receive all events
	c := make(chan map[string]string, 100)
	a.SetEventChannel(c)

	// Check if connected with Asterisk, will send Action "QueueSummary"
	if a.Connected() {
		result, err := a.Action(map[string]string{"Action": "QueueSummary", "ActionID": "Init"})
		// If not error, processing result. Response on Action will follow in defined events.
		// You need to catch them in event channel, DefaultHandler or specified HandlerFunction
		fmt.Println(result, err)
	}

	// Use ActionKV when you need duplicate fields (e.g. multiple Variable)
	if a.Connected() {
		kv := []amigo.KV{
			{Key: "Action", Value: "Originate"},
			{Key: "Channel", Value: "SIP/100"},
			{Key: "Exten", Value: "200"},
			{Key: "Context", Value: "default"},
			{Key: "Priority", Value: "1"},
			{Key: "Variable", Value: "foo=1"},
			{Key: "Variable", Value: "bar=2"},
		}
		result, err := a.ActionKV(kv)
		fmt.Println(result, err)
	}
	
	ch := make(chan bool)
	<-ch
}
```

## SIC!
You should not modify received events, because it can be read in another amigo goroutine.
If you need to modify, you should copy all values to another map and modify it.
