# mianl
免流
/*
    普通免流   例子，只需要修改HTTP/HTTPS代理IP跟模式(可作为wap模式)
*/

#######UDP部分########
httpUDP::udp {
    //如果搭建了cns服务器可以删除下面的//(尽量不要搭建443端口)
    udp_tproxy_listen = 0.0.0.0:6650;
    udp_socks5_listen = 0.0.0.0:1081
    destaddr = 14.215.179.244:443;
    httpMod = tunnel;
    header_host = ;
    encrypt = 
}

tcp::Global {
    tcp_listen = :::6650;
}

//非标准端口转发另行处理
tcpProxy::https_cns_proxy {
    destaddr = 127.0.0.1:1888;
    socks5_client = on;
}
//HTTPS模式
httpMod::tunnel {
    del_line = host;
    set_first = "[M] [H] [V]\r\nHost: [H]\r\nX-T5-Auth: 1962898709\r\nUser-Agent: okhttp/3.11.0 Dalvik/2.1.0 (Linux; U; Android 12; Redmi K40 Build/RKQ1.200826.002) baiduboxapp/11.0.5.12 (Baidu; P1 11)\r\n";
}
//HTTP模式
httpMod::http {
    del_line = host;
    set_first = "[M] http://[H_P][U] [V]\r\nHost: [H_P]\r\nX-T5-Auth: 1962898709\r\nUser-Agent: okhttp/3.11.0 Dalvik/2.1.0 (Linux; U; Android 12; Redmi K40 Build/RKQ1.200826.002) baiduboxapp/11.0.5.12 (Baidu; P1 11)\r\n";
}

tcpProxy::http_proxy {
    //HTTP代理地址
    destaddr = 14.215.179.244:443;
    httpMod = http;
}
tcpProxy::https_proxy {
    //HTTPS代理地址
    destaddr = 14.215.179.244:443;
    tunnelHttpMod = tunnel;
    tunnel_proxy = on;
}

//ssl端口先建立CONNECT连接
tcpAcl::firstConnect {
    tcpProxy = https_proxy;
    matchMode = firstMatch;
    //读取数据后尝试匹配tcpAcl::http模块
    reMatch = http;

    dst_port = 443;
    dst_port = 8000;
    dst_port = 8080;
    dst_port = 8443;
    //continue: dst_port = 8443;
    //continue: dst_port = 33445;

}
//匹配普通http请求
tcpAcl::http {
    tcpProxy = http_proxy;

    continue: method != IS_NOT_HTTP|CONNECT;
    reg_string != WebSocket;
}

//匹配其他请求
tcpAcl::CONNECT {
    tcpProxy = https_cns_proxy;
    dst_port != 0;
}


dns::Global {
    dns_listen = :::6653;
    cachePath = /dev/null;
}
dnsAcl {
    httpMod = http;
    //HTTP代理地址
    destaddr = 14.215.179.244:443;
    header_host = 119.29.29.29;
    query_type = A;
    //query_type = AAAA;
}

//用于接收socks5请求
socks5::recv_socks5 {
    socks5_listen = 0.0.0.0:1081;
    socks5_dns = 127.0.0.1:6653;
    handshake_timeout = 60;
}

Tun {
    tunAddr4 = 10.0.0.1;
    tunAddr6 = fc00::1;
    tunMtu = 1500;
    tunDevice = tunDev;
}
