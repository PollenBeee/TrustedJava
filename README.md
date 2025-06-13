# TrustedJava (TJ) README
## Please refer to the research paper for formal definitions and contact the authors for collaboration opportunities.

## Overview
TrustedJava (TJ) is a programming language designed to enhance object-oriented programming with a novel ownership types system called **owners-as-trusters**. This system addresses the limitations of traditional ownership models by introducing **trusted ownership declassification** with a **neighbourhood relationship**. It allows for dynamic, controlled sharing of objects while maintaining encapsulation and security, mitigating issues like the **tail-gate access problem** where unintended objects gain access to declassified data.

The language extends concepts from ownership types, enabling developers to explicitly declassify objects for dynamic sharing while restricting access to a trusted set of objects defined through a neighbourhood relationship. This README provides an overview of TrustedJava, its key features, syntax, and a sample implementation based on the provided code and the accompanying research paper.

## Key Concepts

### Ownership Types
Ownership types in object-oriented programming enforce encapsulation by restricting object access within an **ownership tree**, a hierarchical structure rooted at the `world` context. Traditional ownership models include:
- **Owners-as-dominators**: Restricts access to an object to only those within its owner's boundary, ensuring strict encapsulation but limiting flexibility for dynamic sharing.
- **Owners-as-modifiers**: Allows read-only access from outside the owner's boundary, weakening encapsulation.
- **Owners-as-downgraders**: Permits dynamic declassification, allowing objects to move up the ownership tree, but risks unintended access by peer objects (tail-gate access problem).

### Owners-as-Trusters
The **owners-as-trusters** model introduces two key mechanisms:
1. **Dynamic Declassification**: Objects can be explicitly declassified to change their dynamic owner, making them visible to objects at a higher level in the ownership tree.
2. **Trusted Neighbourhood**: Access to declassified objects is restricted to a trusted subset of peer objects through a directional **neighbourhood relationship**. This decouples **visibility** (who can see the object) from **accessibility** (who can access it), enhancing security.

### Neighbourhood Relationship
The neighbourhood relationship defines which objects can access a declassified object. It comes in three modes:
- **Per-object mode** (`x neighbours y`): Only object `y` can access `x`'s declassified objects.
- **Per-object subtree mode** (`x neighbours y<#>`): Object `y` and its immediate representation context can access `x`'s declassified objects.
- **Per-object subtree* mode** (`x neighbours y<*>`): Object `y` and its recursive subtree can access `x`'s declassified objects.

### Tail-gate Access Problem
In traditional declassification (e.g., owners-as-downgraders), declassifying an object to a higher level in the ownership tree makes it accessible to all peer objects at that level, including unintended ones (e.g., a `hacker` object in the medical study example). TrustedJava solves this by using the neighbourhood relationship to explicitly specify trusted objects, preventing unauthorized access.

## Syntax
TrustedJava extends Java-like syntax with constructs for ownership, declassification, and neighbourhood relationships. Below is a summary of key syntactic elements (as outlined in the research paper):

### Class and Object Declaration
```java
class C<O> {
    [t_owner][d_owner] C<O> field;
}
```
- `O`: Ownership context (e.g., `this`, `world`, or a named context).
- `t_owner`: Trusted owner (`this` or `?` for abstraction).
- `d_owner`: Dynamic owner, which can change via declassification.

### Declassification and Neighbourhood
```java
declassify e; // Declassifies object e, changing its dynamic owner
x neighbours y; // Establishes a per-object neighbourhood relationship
x neighbours y<#>; // Per-object subtree mode
x neighbours y<*>; // Per-object subtree* mode
```

### Example Syntax from Provided Code
```java
class MedicalRecords<M> {
    object patientRecord : [this][this] PatientRecord<this>;
    function getRecords() : PatientRecord {
        return declassify patientRecord;
    }
}

class WorldOfObjects<world> {
    object hospital : [this][this] Hospital<world>;
    object researcher : [this][this] Researcher<world>;
    object hacker : [this][this] Hacker<world>;
    function main(args : Array(String)) : _ {
        hospital neighbours researcher;
        researcher -> pr; // Now accessible due to neighbourhood
    }
}
```

## Sample Implementation

Below is a sample implementation in TrustedJava, adapted from the provided code, demonstrating the owners-as-trusters model in a medical study scenario. The example illustrates how sensitive patient records can be shared with trusted entities (e.g., a researcher) while preventing access by untrusted entities (e.g., a hacker).

```java
package pack1;

import pack2.Hospital;
import pack2.Doctor;

class PatientRecord<P> {
}

class MedicalRecords<M> {
    object patientRecord : [this][this] PatientRecord<this>;
    function getRecords() : PatientRecord {
        return declassify patientRecord;
    }
}

class Researcher<R> {
}

class Hacker<H> {
}

class WorldOfObjects<world> {
    object hospital : [this][this] Hospital<world>;
    object researcher : [this][this] Researcher<world>;
    object hacker : [this][this] Hacker<world>;

    function main(args : Array(String)) : _ {
        // Establish peer-object relationships
        researcher -> hospital;
        hospital -> researcher;

        // Declassify patient record
        object pr : PatientRecord<this>;
        pr = hospital.getOriginalPatientRecord(); // Access declassified object

        // Access control before neighbourhood
        hospital -> pr; // Allowed: hospital is the trusted owner
        researcher -> pr; // Denied: no neighbourhood yet
        hacker -> pr; // Denied: no neighbourhood

        // Establish neighbourhood
        hospital neighbours researcher;

        // Access control after neighbourhood
        researcher -> pr; // Allowed: researcher is now a neighbour
        hacker -> pr; // Denied: hacker is not a neighbour
    }
}
```

### Explanation of the Example
1. **Classes and Objects**:
   - `PatientRecord`, `MedicalRecords`, `Researcher`, and `Hacker` are defined with ownership contexts.
   - `WorldOfObjects` contains `hospital`, `researcher`, and `hacker` objects, all owned by the `world` context.

2. **Declassification**:
   - The `getRecords` function in `MedicalRecords` declassifies `patientRecord`, changing its dynamic owner to allow access by higher-level objects.

3. **Neighbourhood**:
   - Initially, the `researcher` cannot access the declassified `patientRecord` (`pr`).
   - After `hospital neighbours researcher`, the `researcher` gains access to `pr`.
   - The `hacker` remains unable to access `pr`, as it is not in the neighbourhood.

4. **Access Control**:
   - The `hospital` can access `pr` as its trusted owner.
   - The neighbourhood relationship ensures only trusted objects (`researcher`) gain access, solving the tail-gate access problem.

## Features and Benefits
- **Dynamic Sharing**: Supports runtime declassification for flexible object sharing.
- **Controlled Access**: Neighbourhood relationships restrict access to trusted objects, enhancing security.
- **Decoupled Visibility and Accessibility**: Unlike traditional ownership models, TrustedJava separates what is visible from what is accessible.
- **Mitigation of Tail-gate Access**: Prevents unintended access to declassified objects by untrusted peers.
- **Type Safety**: Formal typing rules ensure safe declassification and neighbourhood relationships.

## Limitations
- **Complexity**: The introduction of trusted owners and neighbourhood relationships adds complexity to program design.
- **Static Analysis**: While dynamic declassification is supported, static verification of neighbourhood relationships may require additional tools.
- **Future Work**: The research paper suggests exploring unanticipated object evolution and delegation logics to further enhance flexibility.

## Installation and Usage
TrustedJava is a theoretical language presented in the research paper. To implement it:
1. **Extend a Java Compiler**: Modify an existing Java compiler to support TrustedJava's syntax for ownership, declassification, and neighbourhood constructs.
2. **Type Checker**: Implement a type checker based on the typing rules in the paper (Tables 2 and 3).
3. **Runtime Environment**: Ensure the runtime supports dynamic owner changes and neighbourhood enforcement.

For a practical implementation, refer to the formal definitions in the research paper and adapt them to a Java-based compiler or interpreter.

## References
The implementation and concepts are based on the research paper:
- Pradeep Kumar D.S and Janardan Misra, "Trusted Ownership Declassification With Neighbourhood," Accenture Technology Labs, Bangalore, India.

Key references from the paper include:
- Clarke D.G., Potter J., and Noble J., "Ownership Types for Flexible Alias Protection," OOPSLA 1998.
- Lu, Y., Potter, J., Xue, J., "Ownership Downgrading for Ownership Types," APLAS 2009.
- Aldrich, J. and Chambers, C., "Ownership Domains: Separating Aliasing Policy from Mechanism," ECOOP 2004.

## Contributing
As TrustedJava is a research concept, contributions can focus on:
- Developing a compiler or interpreter for TrustedJava.
- Extending the type system for additional use cases.
- Integrating with existing Java frameworks for real-world applications.
  
## To run the TrustedJava follow the following:

>> java -jar trustedJava(v1.0).jar <<"input file">> <<"main class name">>

The sample input file is given in the folder pack1.
The language follows Java like syntax. 

Example:

java -jar trustedJava(v1.0).jar "pack1/in.tj" "WorldOfObjects"

To understand the Output Kindly look at the "Log Explanation.pdf", where we have explained a sample log output and the log details.
