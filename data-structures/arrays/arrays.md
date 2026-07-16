# Array

- An array is a fundamental data structure that stores a collection of items or data elements sequentially in memory, where each item is identified by at least one index or key.

### Characteristics of Arrays

- Contiguous Memory: Highly efficient.

- Indexed Access: Constant-time ($O(1)$) retrieval and look-up.

- CPU cache friendly.

### Why CPU Friendly?

- CPUs prefer contiguous memory because it drastically increases speed through hardware optimization.

- Cache Lines: CPUs do not fetch data from RAM one byte at a time. They fetch data in blocks called cache lines (usually 64 bytes).

- Pre-fetching: When you request the first element of an array, the CPU automatically pulls the next several elements into its ultra-fast L1/L2 cache.

- Cache Hit: Accessing subsequent elements takes a fraction of a nanosecond because the data is already inside the CPU cache.

- Cache Miss: Non-contiguous data (like linked lists) forces the CPU to stall and wait for RAM, which is hundreds of times slower.

- Fast Index Calculation: Simple math and no pointers involved like Linked-Lists.

---