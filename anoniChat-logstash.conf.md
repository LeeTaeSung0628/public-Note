```c
input {
  tcp {
    port => 5044
    codec => json_lines
    type => "auction_log"
  }

  tcp {
    port => 5050
    codec => json_lines
    type => "user_log"
  }
}

filter {
  if [type] == "auction_log" {
    if "BidLogDTO" in [message] {
      grok {
        match => {
          "message" => "BidLogDTO\(userId=%{NUMBER:user_id}, postId=%{NUMBER:post_id}, bidAmount=%{NUMBER:bid_amount}, category=%{WORD:category}\)"
        }
      }

      mutate {
        convert => {
          "user_id" => "integer"
          "post_id" => "integer"
          "bid_amount" => "integer"
        }
        remove_field => ["message"]
      }
    } else {
      drop { }
    }
  }

  if [type] == "user_log" {
    if "UserExchangeLogDTO" in [message] {
      grok {
        match => {
           "message" => "UserExchangeLogDTO\(userId=%{NUMBER:user_id}, exchangeAmount=%{NUMBER:exchange_amount}, payType=%{WORD:pay_type}, payStatus=%{WORD:pay_status}\)"
        }
      }

      mutate { 
        remove_field => ["message"]  
      }
    } else {
      drop { }
    }
  }
}

output {
  if [type] == "auction_log" {
    elasticsearch {
      hosts => ["http://elasticsearch:9200"]
      index => "auction_log"
    }
  }

  if [type] == "user_log" {
    elasticsearch {
      hosts => ["http://elasticsearch:9200"]
      index => "user_log"
    }
  }
}
```