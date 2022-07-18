# Zero Downtime in kubernet (k8s)

Zero-downtime tức là ứng dụng không có thời gian chết. Tuy nhiên để đạt được trạng thái zero-downtime thì chúng ta phải kết hợp rất nhiều yếu tố, từ việc code app sao cho đúng, tới việc triển khai thế nào cho chuẩn. 

Zero-downtime phải được thoả mãn trong các case sau:

- `Scale up` ứng dụng, tăng thêm `pod` chạy.
- `Scale down` ứng dụng, giảm số `pod` chạy.
- `Update` ứng dụng bằng phiên bản mới. (upgrade or downgrade version).
- `Update` hạ tầng triển khai (update phần mềm, update phần cứng).

Chúng ta phải áp dụng điều kiện sau để thỏa mãn case tren:

1. Có cơ chế graceful shutdown.
2. Có đủ thời gian move các request sang các pod khác trước khi shutdown `pod` đó bị shutdown. 
3. `Pod` mới được tạo ra phải sẵn sàng xử lý request trước khi join load-balancing.

Từ những điều kiện trên sẽ ra những trường hợp mà ta sẽ thường găp là (Cơ bản):

- Ứng dụng request-response có thời gian phản hồi nhanh, không phụ thuộc instance xử lý (lý tưởng - case trong bài này)
- Ứng dụng req-res có thời gian phản hồi chậm (ví dụ các ứng dụng upload, xử lý dữ liệu lớn,...)
- Ứng dụng req-res phụ thuộc instance xử lý / request phía trước (ví dụ các app frontend SPA khi gọi css/js,...)
- Ứng dụng sử dụng persistent connection (ví dụ ứng dụng sử dụng websocket,...)

Manifest yaml simple:
```
apiVersion: app/v1
kind: deployment
metadata:
  name: hello-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
  spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        imagePullPolicy: Always
        name: hello-app
        ports:
        - containerPort: 8080

---

apiVersion: v1
kind: Service
metadata:
  name: hello-app
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30001
      protocol: TCP
  selector:
    app: hello-app

```

Sử dụng fortio để tạo request:

```fortio load -qps 100 -t 60s "http://157.230.xxx.yyy:30001/" ```

### Readiness check
---
Để giải quyết bài toán Ứng dụng cần thời gian để khởi động, K8S cung cấp cho chúng ta 1 công cụ gọi là Readiness probe. Đây thực chất là 1 cấu hình test. Theo đó K8S sẽ thực hiện kiểm tra Readiness probe định kỳ để check xem ứng dùng đã thật sự sẵn sàng để xử lý request hay chưa, từ đó mới điều hưởng request tới pod.
Để cấu hình Readiness probe chúng ta thêm đoạn manifest sau:

```kubernet
 kind: deloyment
    ...
      containers:
      - image: gcr.io/google-samples/hello-app:2.0
        readinessProbe:
          httpGet:
             path: /
             port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          successThreshold: 1
```
Config ở trên báo cho K8S chạy kiểm tra bằng cách thực hiện 1 request http GET / tới container trên cổng 8080. Interval là 5 giây. Thời gian chờ khởi động 5 giây. Số lần thành công để tính là ready: 1
Bằng việc check Readiness probe, chúng ta đã delay việc forward request tới pod cho tới khi ứng dụng sẵn sàng

### Prestop hook
---
Để giải quyết vấn đề update routing trên K8S cần thời gian, chúng ta sẽ can thiệp vào 1 lifecycle hook của K8S tên là preStop. Prestop hook được gọi theo kiểu synchorous trước khi gửi tín hiệu shutdown tới pod. Việc của preStop hook này đơn giản chỉ là chờ một lát để load balancer ngừng forward request tới pod trước khi shutdown app

```kubernet
containers:
      - image: gcr.io/google-samples/hello-app:2.0
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          successThreshold: 1
        lifecycle:
          preStop:
            exec:
              command: ["/bin/bash", "-c", "sleep 15"]
```
