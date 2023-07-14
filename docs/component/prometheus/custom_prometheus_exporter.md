# 自定义 Prometheus exporter

# 准备
```
安装miniconda
创建虚拟环境
conda create -n exporter  --clone base
安装pyinstaller ,后面打独立程序包使用
pip install pyinstaller -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
```
# 说明

1. exporter 提供以metrics为router的接口
1. 按照prometheus 的数据格式返回数据
# 示例代码
```
import argparse
import json
from http.server import HTTPServer, BaseHTTPRequestHandler
import urllib.request


target_url = ""


class Metrics(BaseHTTPRequestHandler):

    def do_GET(self):
        print(self.path)
        if self.path != "/metrics":
            self.send_response(404)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps({"code": 404, "msg": "url not found"}).encode())
        else:
            self.send_response(200)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            res = urllib.request.urlopen(target_url)
            data = res.read().decode("utf-8")
            #todo(此处根据返回data封装成需要的数据格式返回)
            self.wfile.write(json.dumps({"code": 200, "msg": "success", "data": data}).encode())


if __name__ == '__main__':
    parser = argparse.ArgumentParser(description="yarn exporter")

    #exporter 服务的ip
    parser.add_argument('--inventory', '-i', action='store',
                        required=True,
                        help='localhost ip')
    #exporter 服务的端口
    parser.add_argument('--port', '-p', action='store',
                        required=True,
                        help='exporter port')

    #需要请求的第三方服务的http 接口
    parser.add_argument('--url', '-u', action='store',
                        required=True,
                        help='yarn url')

    args = parser.parse_args()
    target_url = args.url
    host = (args.inventory, int(args.port))
    server = HTTPServer(host, Metrics)
    print("Starting exporter, listen at: %s:%s" % host)
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        pass
    server.server_close()

```
# 制作独立程序
```
pyinstaller -F exporter.py --clean
```
# 测试运行
```
./exporter -i 0.0.0.0 -p 22222 -u http://192.168.0.122:3000/metrics
```