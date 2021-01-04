LRU

```java
package com.example.demo;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @Author fu
 * @Date 2021/1/4 2:44 下午
 **/
public class LRUTest {


    class LinkNode {

        private Node head;
        private Node tail;
        private int capacity;

        private Map<Object, Node> tables = new ConcurrentHashMap<>();

        public Object get(Object key) {
            Object value = null;
            Node node = getValue(key);
            if (node != null) {
                moveHead(node);
                value = node.value;
            }
            return value;
        }

        private void moveHead(Node node) {
            remove(node);
        }

        private Node getValue(Object key) {
            return tables.get(key);
        }
        public void remove(Node node) {
            if (node == head) {
                return;
            } else if (node == tail) {
                tail = node.pre;
                tail.next = null;
            } else {
                node.next.pre = node.pre;
                node.pre.next = node.next;
            }
            node.next = head;
            head.pre = node;
            node.pre = null;
            head = node;
        }
        public void put(Object key, Object value) {
            if (head == null) {
                head = new Node();
                head.key = key;
                head.value = value;
                tail = head;
                tables.put(key, head);
                return;
            }
            Node node = tables.get(key);
            if (node != null) {
                node.value = value;
                remove(node);
            } else {
                Node nNode = new Node();
                nNode.key = key;
                nNode.value = value;
                if (capacity <= tables.size()) {
                    tables.remove(tail.key);
                    tail = tail.pre;
                    tail.next = null;
                }
                tables.put(key, nNode);
                head.pre = nNode;
                nNode.next = head;
                head = nNode;
            }

        }

    }
    static class Node{
        Object key;
        Object value;
        Node pre;
        Node next;
    }

}

```

