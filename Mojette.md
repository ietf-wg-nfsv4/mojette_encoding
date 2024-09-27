# Mojette Transform 
## Introduction
The Mojette Transform is an erasure coding technique that provides fault tolerance for data storage systems by enabling the recovery of lost data blocks. This section describes the integration of the systematic Mojette Transform into the NFS protocol, focusing on encoding and decoding file system blocks, typically sized at 4KB or 8KB.
### Encoding
The Mojette Transform involves the following steps to encode a data block:
1. **Initialization**: Each data block is treated as a 2D grid of data elements (pixels). Typically, a block is structured as a matrix of size $P \times Q$, where $P$ and $Q$ are the dimensions of the grid.
2. **Projections Calculation**: Projections are computed along specific directions defined by pairs of coprime integers $(p_i, q_i)$. Each projection sums the values of the data elements (pixels) along a line defined by these directions. The size of a projection is given by:
$$\text{Size of projection} = (P - 1) \times |q| + (Q - 1) \times |p| + 1$$
For a given projection direction $(p_i, q_i)$, the projection values are calculated as:$$\text{Projection}(b, p_i, q_i) = \sum_{k=0}^{Q-1} \sum_{l=0}^{P-1} \text{Data}(k, l) \cdot \Delta(b - l \cdot p_i + k \cdot q_i)$$
where $\Delta$ is 1 if the argument is zero and 0 otherwise.
### Decoding
Decoding the Mojette Transform is the inverse of encoding, involving the reconstruction of data from projection data to fill an empty grid. This involves solving a system of linear equations defined by the projection differences and the projection directions $(p_i, q_i)$. The algorithm iterates to refine the values of the missing data elements until the original data block is reconstructed.

Data reconstruction is possible if Katz's criterion holds, which was extended to any convex shape.The Katz's criterion specifies that reconstruction is valid if for a given set of $n$ projections along $n$ directions $(p_i,q_i)$ either  $\sum_{i=0}^{n} q_i \ge Q$ or $\sum_{i=0}^{n} p_i \ge P$.
Adjusting the number of lines $Q$ and the projections set allows setting a desired fault-tolerance threshold.

For example a $64\times4$ grid can be decoded by the projection set $\{(0, 1), (1, 1), (2, 1), (3, 1)\}$ as $\sum_{i=1}^{4} q_i = 4$.
### Systematic and Non Systematic Implementations
A systematic code is an error-correcting code where the input data is embedded directly in the encoded output. In contrast, a non-systematic code produces an output that does not contain the original input symbols. The Mojette Transform can be implemented in both ways, allowing it to adapt to various use cases.
### Data Block Representation
In the context of NFS, a data block corresponds to a file system block, which is a contiguous segment of data, typically 4KB or 8KB in size. The Mojette Transform encodes these blocks to ensure data integrity and availability in distributed storage environments.
## Non Systematic Mojette Transform 
### Block Encoding
In the non-systematic version of the Mojette Transform, the original data block is not directly included in the encoded output. Instead, the entire encoded output consists of projections computed from the original data. The number of computed projections $n$ is larger than the number of projections $m$ required to rebuild the initial data. 
### Block Decoding
To decode a file system block that has undergone the **non systematic Mojette Transform**, the following steps are followed:
1. **Identify Available Projections**: Determine which projections are available. A least $m$ projections (what ever they are) out of $n$ must be available. 
2. **Recompute Data Block**: Apply the inverse Mojette Transform to rebuild the original Data.
### Example
Assume a file system block of 4KB is divided into a 128x4 matrix of 128-bit elements. Using the non-systematic Mojette Transform, we compute projections along selected directions, such as $(-2,1)$, $(-1, -1)$ , $(0,1)$, $(1,1)$, $(2,1)$ and $(3,1)$. The original data is not stored directly; instead, the projections are stored.

If a data loss occurs, for instance, if two projections are lost, the missing elements can be recovered by using the remaining projections and solving the inverse problem. Any set of 4 projections among the 6 generated can rebuild exactly the original data block
## Systematic Mojette Transform
### Block Encoding
In the systematic version, the original data block (file system block) is part of the encoded output. Additional projections are calculated to provide redundancy. If $k$ is the number of original data blocks and $n$ is the total number of encoded blocks (including projections), the systematic code will have the first $k$ blocks as the original data and the remaining $n - k$ blocks as projections.
### Block Decoding
To decode a file system block that has undergone the **systematic Mojette Transform**, the following steps are followed:
1. **Identify Missing Data**: Determine which data lines) are missing in the block. Let $e$ be the number of missing lines.
2. **Recompute Projections**: Compute the projections of the available (partial) data blocks. Calculate the differences between the projections of the full data and the partial data.
3. **Combine Existing Data and Recomputed projection**: Recreate the full block of data by combining the existing data with the reconstructed data according to their positions in the block.
### Example
Assume a file system block of 4KB is divided into a $64\times4$ matrix of 128-bit elements. Using the systematic Mojette Transform, we first compute projections along selected directions, such as $(0,1)$, and $(1,1)$. The original 4 blocks of 64 128-bit elements remains part of the encoded data, and the 2 additional projections are stored for redundancy. If a data loss occurs, the missing elements can be recovered by using the projections and solving the inverse problem.
## Conclusion
The Mojette Transform provides two implementations for an efficient and effective way to enhance data reliability by encoding file system blocks with additional projections. These methods ensure that data can be reconstructed even in the presence of failures, thereby enhancing the fault tolerance of the file system. Both have benefits:

**Benefits of Non-Systematic Mojette Transform**:

- **Fast Failure Detection**: By storing only encoded data and having the ability to rebuild the original block from any sufficient subset of projections, the non-systematic encoding implementation offers great flexibility and allows fast failure detection and recovery.
- **Constant Performance**: The non-systematic decoding algorithm is the most performant of all implementations. Although decoding always occurs, the overhead is low, and unlike systematic encoding, performance remains constant regardless of the number of failures.

**Benefits of Systematic Mojette Transform**:

- **Redundancy Reduction**: Systematic Mojette coding reduces redundancy by integrating the original data blocks into the encoded data, unlike non-systematic codes that generate entirely new data from the original.
- **Efficiency**: Fewer projections need to be calculated and stored, reducing both computational and storage overhead.
- **Performance**: Decoding is faster and simpler, especially when some original data blocks are available, enabling quicker data access. However, performance is slightly degraded in the case of failures and depends on the number of failures.

In summary, the Mojette Transform offers robust solutions for data reliability in file systems, balancing redundancy, efficiency, and performance to ensure data integrity and quick recovery from failures.