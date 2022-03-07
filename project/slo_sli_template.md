# API Service

| Category     | SLI | SLO                                                                                                         |
|--------------|-----|-------------------------------------------------------------------------------------------------------------|
| Availability |  Percentage of 200 OK requests in the last n minutes   | 99%                                                                                                         |
| Latency      |  Time taken by 90% of the requests in the last n minutes   | 90% of requests below 100ms                                                                                 |
| Error Budget |  Percentage of 200 OK requests in last n minutes should be >= 80  | Error budget is defined at 20%. This means that 20% of the requests can fail and still be within the budget |
| Throughput   |  Rate with which 200 OK requests are being served in the last n minutes   | 5 RPS indicates the application is functioning                                                              |
