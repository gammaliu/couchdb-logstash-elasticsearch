# couchdb-logstash-elasticsearch
Show how to config this software stack

This is the basic logstash.conf for the " CouchDB -> Logstash -> Elasticsearch " Stack, 
Where the JSON document has same structure between couchdb and elasticsearch...

```ruby
input { 
        couchdb_changes {
                db => "meizi"
                #host => "localhost"
                host => "127.0.0.1"
                port => 5984
                initial_sequence => 0
        }
}




filter {
        # 1)
        mutate {
                add_field => {
                        "[@metadata][my_id]" => "%{[@metadata][_id]}"
                }
        }
        # 2)
        if "delete" in [@metadata][action] {
                mutate { add_field => { "[@metadata][my_action]" => "delete" } }
        }
        else {
                mutate { add_field => { "[@metadata][my_action]" => "index" } }
        }
        # 3)
        if ![doc][Metadata] {
                mutate { add_field => { "[@metadata][my_type]" => "video" } }
        }
        else if [doc][Metadata][Program]{
                mutate { add_field => { "[@metadata][my_type]" => "program" } }
        }
        else if [doc][Metadata][Sequence] {
                mutate { add_field => { "[@metadata][my_type]" => "sequence" } }
        }
        else if [doc][Metadata][Scene] {
                mutate { add_field => { "[@metadata][my_type]" => "scene" } }
        }
        else if [doc][Metadata][Shot] {
                mutate { add_field => { "[@metadata][my_type]" => "shot" } }
        }
        else {
                mutate { add_field => { "[@metadata][my_type]" => "video" } }
        }
        # 4)
        if [doc][parent] {
                mutate { add_field => { "[@metadata][parent_id]" => "%{[doc][parent]}" } }
        }
        else {
                mutate { add_field => { "[@metadata][parent_id]" => "not_defined" } }
        }
        # 5)
        if [doc] {
                ruby {
                        code => '
                                event["doc"].each{ |k,v| event["#{k}"] = v }
                                '
                }
        }
        # 6)
        if [doc] {
                mutate { remove_field => [ "[doc]" ] }
        }
        if [doc_as_upsert] {
                mutate { remove_field => [ "[doc_as_upsert]" ] }
        }
        if [@version] {
                mutate { remove_field => [ "[@version]" ] }
        }
}

output {
        elasticsearch {
                codec => rubydebug { metadata => true }
                #hosts => "localhost:9200"
                hosts => "127.0.0.1:9200"
                index => "meizi"
                document_type => "%{[@metadata][my_type]}"
                document_id => "%{[@metadata][my_id]}"
                action => "%{[@metadata][my_action]}" 
        }
}
```
