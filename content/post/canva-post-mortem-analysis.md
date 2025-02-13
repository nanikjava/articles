## Report

Analysis based on the published [post mortem report by Canva](https://www.canva.dev/blog/engineering/canva-incident-report-api-gateway-outage/) 

The API Gateway cluster failure was caused by multiple factors, including:

* A **software deployment** related to Canva's editor, which contributed to system instability.
* A **locking issue**, potentially affecting **resource availability** and **request handling**.
* **Network issues** in Cloudflare, which further disrupted connectivity and compounded the failure.


## User and Canva Editor

* The Canva Editor is the main UI application that users interact with.
* **Single Page Application (SPA)** (assumed built using **TypeScript** and/or **JavaScript**).
* Symptons:
    * The page appears to be continuously loading, seemingly waiting for assets to be downloaded.
    * Users may keep waiting, expecting the application to load eventually.
    * Users must have been impatience and refreshing the browser repeatedly.

## Cloudflare (CF)

* CF uses cache stream (**Concurrent Streaming Acceleration**) to optimize caching performance. The feature consolidate all user's request requesting the same assets to be batch together and once assets are obtained from origin all the request will get the response.
* Since request are not rejected (it's put on hold by CF) so no request errors are logged so not alert are triggered.
* Even as there are 270,000 requests waiting in CF queue for the assets there is only 1 connection from CF going to the origin server to get the assets, as the assets has not been cache. Once the assets are cached it is delivered to all of the 270,000 waiting request.

![Cloudflare](/static/media/canva/cf.png)


## API Gateway

* The 270,000 user devices have the latest code running. The number of request hitting the API gateway now triple peak load (1.5 million request/sec).
* The running gateways were not able to process the requests successfully as newly introduced telemetry SDK code introduced a new bug that created lock contention.
* The lock contention bug magnified the issue as requests are now failing because requests cannot be processed by the available running gateway.
* Auto scaling should be able to solve the problem but it was not able to scale up quickly as the number of requests are overhelming the infrastructure.


![Gateway](/static/media/canva/gateway.png)

## Lock Contention

What is `lock contention`:

```
this refers to a scenario in concurrent programming where multiple threads or processes are contending for access to a shared resource, and there is a high likelihood of conflicts or contention 
events occurring. 
```

In the context of the Telemtry SDK there is a shared data structure that probably were not intended for concurrent access but the code that uses
the data are accessing it concurrently creating a lock contention.

The post mortem highlight the part where the gateway is indeed doing it's job in an event loop and code running in the event loop should not
perform any blocking operation, but in this case it is performing blocking operation producing lock contention.

```
The API Gateways use an event loop model, where code running on event loop threads must not perform any blocking operations. 
```

![Cloudflare](/static/media/canva/lock_contention.png)

## Key Takeaways

* Understanding the SDK (even if it is an internally developed) limitation is crucial in using it correctly.
* Designing concurrent application is hard so duty of care should be taken to perform proper testing.
* **Concurrent performance testing** should be part of the release process. Having standard performance baseline enables to 
compare how the application is performing and this can assist in picking up anomaly.