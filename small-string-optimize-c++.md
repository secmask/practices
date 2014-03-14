#C++ Small String Optimize
Most STL implement use a technical call Small String Optimize(SSO), that store data direct in the string object instead of alloc storage data from heap, which will run faster(no malloc/free needed). Follow is proof of concept how does it implement:

    class string {
    public:
        // all 83 member functions
    private:
        size_type m_size;
        union {
            class {
                // This is probably better designed as an array-like class
                std::unique_ptr<char[]> m_data;
                size_type m_capacity;
            } m_large;
            std::array<char, sizeof(m_large)> m_small;
        };
    };