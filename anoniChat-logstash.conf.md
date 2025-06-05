```c
input {

	beats {
	    port => 5044
	}
	  
	tcp {
	    port => 5000
	    codec => json_lines
	    type => "main_log"
	}
}

filter {
	if [type] == "main_log" {
		grok {
			match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{DATA:thread}\] %{LOGLEVEL:loglevel} %{DATA:logger} - %{GREEDYDATA:logmessage}" }
		}
	}
}

output {
	if [type] == "auction_log" {
	    elasticsearch {
	      hosts => ["http://elasticsearch:9200"]
	      index => "main_log"
		}
	}
}
```