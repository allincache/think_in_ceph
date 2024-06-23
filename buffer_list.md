**simplified buffer::list code**

</br>

```cpp
// Simplified Ceph Buffer Management Code

#include <iostream>
#include <memory>
#include <vector>

namespace ceph {
namespace buffer {

// Abstract buffer class
class raw {
public:
    virtual ~raw() {}
    virtual char* data() = 0;
    virtual size_t length() const = 0;
};

// Concrete static buffer implementation
class raw_static : public raw {
    char* buf;
    size_t len;
public:
    raw_static(char* buffer, size_t length) : buf(buffer), len(length) {}
    char* data() override { return buf; }
    size_t length() const override { return len; }
};

// Buffer pointer class
class ptr {
    std::shared_ptr<raw> raw_ptr;
    size_t offset, length;
public:
    ptr(std::shared_ptr<raw> r, size_t o, size_t l) : raw_ptr(r), offset(o), length(l) {}
    char* c_str() { return raw_ptr->data() + offset; }
    size_t length() const { return length; }
};

// Iterator for the buffer list
class list::iterator {
    std::vector<ptr>::iterator it;
public:
    iterator(std::vector<ptr>::iterator it) : it(it) {}

    char* operator*() {
        return (*it).c_str();
    }

    iterator& operator++() {
        ++it;
        return *this;
    }

    bool operator==(const iterator& other) const {
        return it == other.it;
    }

    bool operator!=(const iterator& other) const {
        return it != other.it;
    }
};

// Buffer list class
class list {
    std::vector<ptr> buffers;
    size_t total_length;
public:
    // Iterator-related functions
    iterator begin() { return iterator(buffers.begin()); }
    iterator end() { return iterator(buffers.end()); }

    void push_back(ptr bp) {
        buffers.push_back(bp);
        total_length += bp.length();
    }

    size_t length() const { return total_length; }

    // Operator overloading to allow usage like std::string
    char operator[](size_t index) const {
        size_t offset = 0;
        for (const auto& bp : buffers) {
            if (index < offset + bp.length()) {
                return bp.c_str()[index - offset];
            }
            offset += bp.length();
        }
        throw std::out_of_range("Index out of range");
    }

    // ... Other methods such as append, erase, etc.
};

// Example usage
int main() {
    char static_buffer[10] = "Hello";
    auto static_raw = std::make_shared<raw_static>(static_buffer, 5);
    ptr static_ptr(static_raw, 0, 5);

    list buffer_list;
    buffer_list.push_back(static_ptr);

    std::cout << "Buffer list length: " << buffer_list.length() << std::endl;
    for (auto it = buffer_list.begin(); it != buffer_list.end(); ++it) {
        std::cout << *it << std::endl; // Note: Assuming data is null-terminated
    }

    // Using operator overloading
    std::cout << "First character: " << buffer_list[0] << std::endl;

    return 0;
}

} // namespace buffer
} // namespace ceph
```