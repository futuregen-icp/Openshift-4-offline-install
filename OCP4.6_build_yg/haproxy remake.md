acl yo4_ocp3_api req_ssl_sni -m end -i yo4.ocp3.fu.igotit.co.kr
   use_backend yo4_ocp3_api if yo4_ocp3_api

backend yo4_ocp3_api
   #balance leastconn
    mode tcp 
    server bootstrap 192.0.0.19:6443 check 
    server master01 192.0.0.21:6443 check 
    server master02 192.0.0.22:6443 check 
    server master03 192.0.0.23:6443 check 
    
    
    
    acl yo4_ocp3_mc req_ssl_sni -m end -i yo4.ocp3.fu.igotit.co.kr
    use_backend backend yo4_ocp3_mc if backend yo4_ocp3_mc
    
backend yo4_ocp3_mc
    #balance leastconn
    balance source
    mode tcp
    server bootstrap 192.0.0.19:22623 check 
    server master01 192.0.0.21:22623 check 
    server master02 192.0.0.22:22623 check 
    server master03 192.0.0.23:22623 check 


acl yo4_ocp3_https req_ssl_sni -m end -i apps.yo4.ocp3.fu.igotit.co.kr
use_backend yo4_ocp3_https if yo4_ocp3_https

backend yo4_ocp3_https
    balance leastconn
    # balance source
    mode tcp
    #option tcplog
    server worker01 192.0.0.121:443 check
    server worker02 192.0.0.122:443 check
    
    

    acl yo4_ocp3_http hdr_end(host) -i apps.yo4.ocp3.fu.igotit.co.kr
    use_backend yo4_ocp3_http if yo4_ocp3_http

backend yo4_ocp3_http
    balance source
    # mode tcp
    # option tcplog
    server worker01 192.0.0.121:80 check
    server worker02 192.0.0.122:80 check
