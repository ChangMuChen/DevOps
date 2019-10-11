ceph获取admin的key:

ceph auth get-key client.admin |base64

备注：因为k8s中的secret使用的是base64加密,所以上面在获取key后用base64加了一下密
