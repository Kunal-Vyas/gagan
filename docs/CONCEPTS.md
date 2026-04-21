# Framework

1. **Gagan** represents any field as a matrix of matrices.
2. Each individual constituent matrix represents a single point in the field — for example, in a spacetime field, a constituent represents a point in spacetime.
3. Every constituent matrix has the same number of elements as the number of dimensions in the field. For example:
   - A one-dimensional field with a single constituent could be represented as `[[1]]`.
   - A point in four-dimensional spacetime could be represented as `[[0, 1, 0, 0]]`.
4. Each element in a constituent matrix holds a value of either `0` or `1`. The meaning of these values is predefined by the model (e.g., `0` might represent an "inactive" state and `1` an "active" state along a given dimension). This binary constraint keeps the analysis simple and is well-suited to the underlying computational approach.
5. The constituent matrices can be configured in the same number of ways as the number of dimensions in the field. Each configuration corresponds to activating a different dimension, marked by a superscript label on the matrix. For example, in a two-dimensional field, the two possible configurations are `[[0, 1]]¹` (oriented toward dimension 1) and `[[0, 1]]²` (oriented toward dimension 2). The values remain the same — only the configuration label changes. See the [spacetime implementation](./implementations/4D-SPACETIME.md) for a concrete example using dimension-specific superscripts (ˣ, ʸ, ᶻ, ᵗ).
6. Every configuration of the constituent matrices is complementary to the others — no two configurations activate the same dimension. The interaction between field constituents that share the same values but differ in configuration can be predefined by the model.
7. Analysis begins with a set of predefined initial conditions. Values for the constituent matrices are then generated stochastically (each element is randomly assigned `0` or `1`).
8. Events generated for each assigned value are monitored and published. These events can be subscribed to and reacted to using event-based analysis.
9. Under predefined conditions — such as threshold counts, spatial proximity, or temporal patterns — new constituents can be created or existing ones removed from the field. This can be done as part of the event-based analysis, alongside other actions.