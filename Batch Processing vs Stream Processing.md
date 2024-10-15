
### Batch Processing
- process data in batches
- data is collected over a period of time and then it is processed in a single finite batch
- Popular frameworks - Apache Hadoop and Apache Spark
- Use cases 
	- Monthly billing report
	- End of day report
- Advantages
	- Efficient resource utilization as it processes data in batches thus reducing the overhead of frequent data loading and processing
	- can be horizontally scalable by distributing the load across multiple machines
	- fault tolerant
	- more cost effective compared to real time processing
	- High throughput
- Disadvantages
	- Delayed processing as large volume of data is processed in batches
	- High latency as batch processing introduces latency between data collection and processing, as data is processed in batches at scheduled intervals
	- Resource intensive

### Stream Processing
- process data in real time as in when it arrives
- you can achieve this using kafka
- Use cases
	- chat application
	- Live data feeds
	- online gaming
- Advantages
	- low latency as data is processed as in when it arrives
	- real time processing
	- horizontally scalable
- Disadvantages
	- Higher cost - due to need for more powerful systems and continuous processing
	- Potential for data loss