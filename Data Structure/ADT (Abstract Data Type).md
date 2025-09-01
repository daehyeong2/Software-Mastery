배열 리스트 추상 자료형 코드 (Python)
```python
class ArrayList:  
    def __init__(self, capacity=100):  
        self.capacity = capacity  
        self.array = [None] * capacity  
        self.size = 0  
    def isEmpty(self):  
        return self.size == 0  
    def isFull(self):  
        return self.size == self.capacity  
    def insert(self, pos, e):  
        if not self.isFull() and 0 <= pos <= self.size:  
            for i in range(self.size, pos, -1):  
                self.array[i] = self.array[i-1]  
            self.array[pos] = e  
            self.size += 1  
        else:  
            pass  
    def delete(self, pos):  
        if not self.isEmpty() and 0 <= pos < self.size:  
            e = self.array[pos]  
            for i in range(pos, self.size-1):  
                self.array[i] = self.array[i+1]  
            self.size -= 1  
            return e  
        else: pass  
    def getEntry(self, pos):  
        if 0 <= pos <= self.size:  
            return self.array[pos]  
        else: return None  
    def __str__(self):  
        return str(self.array[0:self.size])  
  
if __name__ == "__main__":  
    L = ArrayList(50)  
  
    print("최초   ", L)  
    L.insert(0, 10)  
    L.insert(0, 20)  
    L.insert(1, 30)  
    L.insert(L.size, 40)  
    L.insert(2, 50)  
    print("삽입x5", L)  
    L.delete(2)  
    print("삭제(2)", L)  
    L.delete(L.size-1)  
    print("삭제(E)", L)  
    L.delete(0)  
    print("삭제(0)", L)
```