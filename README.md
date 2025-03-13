# data-sync-manager

Creating a complete Python program for a data synchronization manager that automates real-time data synchronization requires careful consideration of various aspects, including the design architecture, error handling, and system interoperability. Below is a simple example of a Python program that simulates the synchronization of data between two systems (System A and System B) using a message queue to manage real-time updates. This example uses Python's built-in `threading` library to simulate real-time behavior and `queue.Queue` for message handling.

```python
import threading
import queue
import time
import random

class DataSystem:
    def __init__(self, name):
        self.name = name
        self.data_store = {}
        self.lock = threading.Lock()

    def update_data(self, key, value):
        with self.lock:
            self.data_store[key] = value
            print(f"{self.name} updated key: {key}, value: {value}")

    def get_data(self, key):
        with self.lock:
            return self.data_store.get(key, None)


class DataSyncManager:
    def __init__(self, system_a, system_b):
        self.system_a = system_a
        self.system_b = system_b
        self.queue_a_to_b = queue.Queue()
        self.queue_b_to_a = queue.Queue()
        self.stop_event = threading.Event()

    def sync_data(self):
        print("Starting data synchronization...")
        # Thread for synchronizing data from System A to System B
        def sync_a_to_b():
            while not self.stop_event.is_set():
                try:
                    key, value = self.queue_a_to_b.get(timeout=1)
                    self.system_b.update_data(key, value)
                except queue.Empty:
                    continue

        # Thread for synchronizing data from System B to System A
        def sync_b_to_a():
            while not self.stop_event.is_set():
                try:
                    key, value = self.queue_b_to_a.get(timeout=1)
                    self.system_a.update_data(key, value)
                except queue.Empty:
                    continue

        threading.Thread(target=sync_a_to_b, daemon=True).start()
        threading.Thread(target=sync_b_to_a, daemon=True).start()

    def update_system_a(self, key, value):
        print(f"Request to update System A: key = {key}, value = {value}")
        self.system_a.update_data(key, value)
        self.queue_a_to_b.put((key, value))  # Send update to System B

    def update_system_b(self, key, value):
        print(f"Request to update System B: key = {key}, value = {value}")
        self.system_b.update_data(key, value)
        self.queue_b_to_a.put((key, value))  # Send update to System A

    def stop_sync(self):
        self.stop_event.set()
        print("Stopping data synchronization...")


# Simulated client usage
if __name__ == "__main__":
    system_a = DataSystem("System A")
    system_b = DataSystem("System B")

    manager = DataSyncManager(system_a, system_b)
    manager.sync_data()

    try:
        # Simulate updates in System A and System B
        for _ in range(5):
            time.sleep(random.uniform(0.5, 1.5))
            if random.choice([True, False]):
                manager.update_system_a(f"key{random.randint(1, 5)}", random.randint(100, 200))
            else:
                manager.update_system_b(f"key{random.randint(1, 5)}", random.randint(100, 200))
    except KeyboardInterrupt:
        print("Terminating synchronization due to user interrupt.")

    # Stop the synchronization threads gracefully
    manager.stop_sync()
    time.sleep(2)  # Allow time for threads to complete
```

### Explanation:

1. **DataSystem Class**: Represents a generic system with data storage. It uses a dictionary to store key-value pairs and a lock to handle thread-safe operations.

2. **DataSyncManager Class**: Manages synchronization between two `DataSystem` instances using two separate queues for each direction of synchronization. It also handles starting and stopping of sync operations using threading.

3. **Threads**: Two threads are used to handle real-time updates between System A and System B, ensuring data consistency across both systems.

4. **Error Handling**: The program uses `queue.Empty` exception handling to manage occasional queue read timeouts gracefully.

5. **Simulated Client**: Demonstrates how to use the `DataSyncManager` to perform update operations and synchronization.

This simple prototype can be extended to handle more sophisticated error handling, retry logic, and integration with real systems or databases.