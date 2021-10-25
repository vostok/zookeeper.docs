# Basics

ZooKeeper cluster consists of a tree-like structure of nodes with `/` as its root. There are two types of nodes: persistent and ephemeral. Once created, persistent node will exist until explicitly deleted. On the other hand, lifetime of an ephemeral node is limited to that of a client-server session within which this node was created.

Only persistent nodes can have children. Any node can also contain (a small amount of) arbitrary bytes as its data.

ZooKeeper client holds a session with servers in cluster that is described by _session id_ and _session timeout_. Client constantly queries servers to keep the session alive and receive notifications. When backend does not hear anything from a client within the whole session timeout, it considers such session expired.

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}

{% content-ref url="creating-a-client.md" %}
[creating-a-client.md](creating-a-client.md)
{% endcontent-ref %}

{% content-ref url="connecting-to-a-zookeeper-cluster.md" %}
[connecting-to-a-zookeeper-cluster.md](connecting-to-a-zookeeper-cluster.md)
{% endcontent-ref %}

{% content-ref url="using-client.md" %}
[using-client.md](using-client.md)
{% endcontent-ref %}

{% content-ref url="limitations.md" %}
[limitations.md](limitations.md)
{% endcontent-ref %}

