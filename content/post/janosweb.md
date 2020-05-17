---
date: "2020-03-08"
title: "Janos Web Utility"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---

Reference: https://github.com/janos/web



├── client
│   ├── api
│   └── http
├── file-server
├── log
│   └── access
├── maintenance
├── minifier
├── recovery
├── server
├── servers
│   ├── grpc
│   │   └── internal
│   │       └── hello
│   ├── http
│   └── quic
└── templates

Framework revolves around Handlers


Samples
-------
file-server
	* hasher.go 
		- hashing functions
	* options.go
		- contains struct defining handler and other http server related options
		- contains global var for handler such as DefaultNotFoundHandler, DefaultForbiddenHandler and DefaultInternalServerErrorHandler
	* server.go
		- HashedPath --> create hashed path from a string
recovery
	* recovery.go
		- provide handler and other utility for handling 'panic' situation in http server environment
server
	* internal.go
		- provide lots of internal handlers for troubleshooting - /status, /data, /debug/pprof. The handler are triggered when specify the port in  Options.ListenInternal
	* handlers.go
		- 
servers/http
	* http.go
		- code that takes care of http related functionality - servers, request, handlers, etc

			package main

			import (
				"fmt"
				"net/http"
				"resenje.org/logging"
				recovery "resenje.org/recovery"
				s "resenje.org/web/server"
			)

			var (
				responseBody = "----- response body -----"
				handler      = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
					fmt.Fprint(w, responseBody)
				})
			)


			func main() {
				rs := &recovery.Service{}
				logger, _ := logging.GetLogger("default")
				logger.SetLevel(8)

				o := s.Options{
					Name:               "servertest",
					Version:            "1.0",
					BuildInfo:          "buildinfo",
					ListenInternal:     "localhost:9200",
					ListenInternalTLS:  "",
					InternalTLSCert:    "",
					InternalTLSKey:     "",
					ACMECertsDir:       "",
					ACMECertsEmail:     "",
					EmailService:       nil,
					RecoveryService:    rs,
					MaintenanceService: nil,
					Logger:logger,
				}

				ho := s.HTTPOptions{
					Handlers:     nil,
					Name:         "",
					Listen:       "localhost:9000",
					ListenTLS:    "",
					TLSCerts:     nil,
					IdleTimeout:  0,
					ReadTimeout:  0,
					WriteTimeout: 0,
				}
				ho.SetHandler(handler)

				ss, _ := s.New(o)
				ss.WithHTTP(ho)
				ss.Serve()

				select {}
			}





