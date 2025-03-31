# E-commerce Application Troubleshooting Guide

This guide provides detailed troubleshooting steps for common issues in the e-commerce application deployment.


## Frontend Accessibility Issues

### Symptoms
- Frontend service not accessible externally
- 404/503 errors when accessing the application URL
- Ingress not routing traffic correctly

### Diagnostic Steps

1. **Check Ingress Status**:
```bash
# Verify Ingress configuration
kubectl get ingress frontend-ingress -n ecommerce -o yaml

# Check Ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

# Verify Ingress controller is running
kubectl get pods -n ingress-nginx
```

2. **Verify Service Configuration**:
```bash
# Check service status
kubectl get svc frontend -n ecommerce

# Test service DNS resolution
kubectl run temp-dns-test --rm -i --tty --image=busybox -- nslookup frontend.ecommerce.svc.cluster.local

# Verify service endpoints
kubectl get endpoints frontend -n ecommerce
```

3. **Check Pod Health**:
```bash
# Verify pod status
kubectl get pods -n ecommerce -l app=frontend

# Check pod logs
kubectl logs -l app=frontend -n ecommerce

# Check pod events
kubectl describe pod -l app=frontend -n ecommerce
```

### Common Solutions

1. **SSL/TLS Issues**:
```bash
# Verify TLS secret exists
kubectl get secret frontend-tls -n ecommerce



## Backend-Database Connectivity Issues

### Symptoms
- Intermittent connection failures
- Timeout errors in backend logs
- MongoDB connection refused errors

### Diagnostic Steps

1. **Check MongoDB Cluster Status**:
```bash
# Verify MongoDB pods
kubectl get pods -l app=mongodb -n ecommerce

# Check MongoDB statefulset status
kubectl get statefulset mongodb -n ecommerce

# Check MongoDB logs
kubectl logs -l app=mongodb -n ecommerce
```

2. **Verify Network Connectivity**:
```bash
# Test MongoDB connectivity from backend
kubectl exec -it $(kubectl get pod -l app=backend -n ecommerce -o jsonpath='{.items[0].metadata.name}') -n ecommerce -- nc -zv mongodb-service 27017

# Check DNS resolution
kubectl exec -it $(kubectl get pod -l app=backend -n ecommerce -o jsonpath='{.items[0].metadata.name}') -n ecommerce -- nslookup mongodb-service
```

3. **Monitor Connection Metrics**:
```bash
# Check MongoDB metrics in Prometheus
kubectl port-forward svc/prometheus 9090:9090 -n monitoring
# Access: http://localhost:9090
# Query: mongodb_connections_current
```

### Symptoms
- Slow order processing
- Messages stuck in RabbitMQ queues
- Increased latency in order status updates

### Diagnostic Steps

1. **Check RabbitMQ Status**:
```bash
# Verify RabbitMQ cluster status
kubectl exec -it rabbitmq-0 -n ecommerce -- rabbitmqctl cluster_status

# Check queue metrics
kubectl exec -it rabbitmq-0 -n ecommerce -- rabbitmqctl list_queues name messages consumers

# Monitor queue rate
kubectl exec -it rabbitmq-0 -n ecommerce -- rabbitmqctl list_queues name messages_ready messages_unacknowledged message_bytes consumers
```

2. **Analyze Consumer Health**:
```bash
# Check consumer logs
kubectl logs -l app=backend -n ecommerce --tail=100

# Monitor consumer metrics
kubectl exec -it rabbitmq-0 -n ecommerce -- rabbitmqctl list_consumers
```

3. **Performance Metrics**:
```bash
# Check RabbitMQ metrics in Prometheus
# Query examples:
# - rabbitmq_queue_messages_ready
# - rabbitmq_queue_messages_published_total
# - rabbitmq_queue_messages_delivered_total
```

## Support Escalation

If issues persist after following these troubleshooting steps:

1. Collect diagnostic information:
```bash
# Collect all logs
kubectl logs -l app=frontend -n ecommerce > frontend-logs.txt
kubectl logs -l app=backend -n ecommerce > backend-logs.txt
kubectl logs -l app=mongodb -n ecommerce > mongodb-logs.txt
kubectl logs -l app=rabbitmq -n ecommerce > rabbitmq-logs.txt

# Get resource descriptions
kubectl describe all -n ecommerce > resource-description.txt
```

---

