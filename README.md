# golang-single-flight-sample

This repository is for demo how to use single-flight package to reduce multiple concurrency duplicate request 

## diagram

1. original problem

![duplicated-multiple-requests.png](duplicated-multiple-requests.png)

2. reduce multiple duplicated requests with single flight

![single-flight.png](single-flight.png)

## concept

use in-memory map structure to store data in time frame with certain key

## code implementation

```golang
package main

import (
	"fmt"
	"net/http"
	"sync"

	"golang.org/x/sync/singleflight"
)

var count int = 0
var lock sync.Mutex = sync.Mutex{}

func getData() (interface{}, error) {
	lock.Lock()
	count++
	lock.Unlock()
	return http.Get("https://google.com")
}
func getDataSingleFlight(g *singleflight.Group, wg *sync.WaitGroup) error {
	defer wg.Done()
	v, err, shared := g.Do("get-google", getData)
	if err != nil {
		return err
	}
	res := v.(*http.Response)
	fmt.Printf("status: %d, shared: %v\n", res.StatusCode, shared)
	return nil
}
func main() {
	var wg sync.WaitGroup
	var g singleflight.Group
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go getDataSingleFlight(&g, &wg)
	}
	wg.Wait()
	fmt.Printf("original function was called %d times\n", count)
}
```