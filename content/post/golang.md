---
date: "2019-09-01"
title: "Golang"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---


Multi-Threading 
---------------
* Remember using 'channel'
	- If we close the channel and we tryto read data (even AFTER reading the data) no exception thrown

		func main(){
			threadChannel := make(chan string,1) // size chosen to ensure it never gets filled up completely.


			go func() {
				threadChannel <- "TESTING from function 1"
				defer close(threadChannel)
			}()

			fmt.Println("Obtained  1" , <-threadChannel)
			time.Sleep(1000 * time.Millisecond)
			fmt.Println("----------------")
			fmt.Println("Obtained  2" , <-threadChannel)
		}


	- If we DO NOT close the channel and try to read data (AFTER reading the data) deadlock exception will be thrown


		func main(){
			threadChannel := make(chan string,1) // size chosen to ensure it never gets filled up completely.


			go func() {
				threadChannel <- "TESTING from function 1"
				//defer close(threadChannel)
			}()

			fmt.Println("Obtained  1" , <-threadChannel)
			time.Sleep(1000 * time.Millisecond)
			fmt.Println("----------------")
			fmt.Println("Obtained  2" , <-threadChannel)
		}



