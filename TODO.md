## qnap routing

- [ ] routing setup
- [ ] advertise routing
- [x] squid proxy

## qnap and mobile data issue

- [x] root cause
- [ ] better report

## docker dns issue 

- [x] found and fix: https://github.com/moby/moby/issues/19474#issuecomment-1903992627

## mdns issue

- [x] Local dns keeps failing. A wrong method was used (`fail!` instead of `fail`). The inresting thing is that the daemon didn't report good message. The wrong method is only seen when debug mode is turned on.! 

```
Jan 24 20:04:16 toshiba.google.com ruby[979744]: /usr/lib/ruby/gems/3.0.0/gems/rubydns-0.6.7/lib/rubydns/handler.rb:137:in `unbind': Unprocessed data remaining (374 bytes unprocessed) (RubyDNS::LengthError)
Jan 24 20:04:16 toshiba.google.com ruby[979744]:         from /usr/lib/ruby/gems/3.0.0/gems/eventmachine-1.0.9.1/lib/eventmachine.rb:1470:in `event_callback'
Jan 24 20:04:16 toshiba.google.com ruby[979744]:         from /usr/lib/ruby/gems/3.0.0/gems/eventmachine-1.0.9.1/lib/eventmachine.rb:193:in `run_machine'
Jan 24 20:04:16 toshiba.google.com ruby[979744]:         from /usr/lib/ruby/gems/3.0.0/gems/eventmachine-1.0.9.1/lib/eventmachine.rb:193:in `run'
Jan 24 20:04:16 toshiba.google.com ruby[979744]:         from /usr/lib/ruby/gems/3.0.0/gems/rubydns-0.6.7/lib/rubydns.rb:73:in `run_server'
Jan 24 20:04:16 toshiba.google.com ruby[979744]:         from /root/dns/main.rb:69:in `<main>'
```
