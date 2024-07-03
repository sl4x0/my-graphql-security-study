## `The Language`

- **Operation Type**
    - Defines the method of interaction with the server.
    - Types: Query, Mutation, Subscription.
- **Operation Name**
    - Arbitrary client-created label.
    - Provides a unique name to an operation.
- **Top-level Field**
    - Function that returns a single unit of information or object.
    - May contain nested fields.
- **Argument (of a top-level field)**
    - Parameter name for sending information to a field.
    - Tailors the behavior and results of the field.
- **Value**
    - Data related to an argument sent to a field.
- **Field**
    - Nested function returning a single unit of information or object.
    - Can be nested within other fields.
- **Directive**
    - Feature to decorate fields.
    - Changes validation or execution behavior.
- **Argument (of a directive)**
    - Parameter name for sending information to a directive.
    - Tailors the behavior and results of the directive.
- **Argument (of a field)**
    - Parameter name for sending information to a nested field.
    - Tailors the behavior and results of the nested field.

### Example Query Components

```graphql
query GetBooks($author: String) {    # Operation type: query, Operation name: GetBooks
  books(author: $author) {           # Top-level field: books, Argument: author
    title                            # Field: title
    publication                      # Field: publication
  }
}
```

## `Queries, Mutations, and Subscriptions`

### `Mutations`

- **Purpose**: Modify data.
- **Example**:
    
    ```graphql
    mutation {
      editPaste(id: 1, content: "My first mutation!") {
        paste {
          id
          title
          content
        }
      }
    }
    ```
    
- **Key Points**:
    - **Mutation Keyword**: `mutation`
    - **Top-level Field**: `editPaste`
    - **Arguments**: `id`, `content`
    - **Nested Fields**: `paste { id, title, content }`
- **Note**: Avoid using queries for data writes; stick to mutations.

### `Subscriptions`

- **Purpose**: Retrieve real-time data and receive updates from the server.
- **Protocol**: Commonly uses WebSocket, but other protocols can be used.
- **WebSocket Handshake**: Upgrades HTTP connection to WebSocket.
    - **Client Request**:
        
        ```
        GET /subscriptions HTTP/1.1
        Host: 0.0.0.0:5013
        Connection: Upgrade
        Upgrade: websocket
        Sec-WebSocket-Version: 13
        Sec-WebSocket-Key: MV5U83GH1UG8AlEb18lHiA==
        ```
        
    - **Server Response**:
        
        ```
        HTTP/1.1 101 Switching Protocols
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Accept: aRnlpG8XwzRHPVxYmGVdqJv3D7U=
        ```
        
- **Example Subscription**:
    
    ```graphql
    subscription {
      paste {
        id
        title
        content
      }
    }
    ```
    
    - **Keyword**: `subscription`
    - **Top-level Field**: `paste`
    - **Nested Fields**: `id`, `title`, `content`
- **Usage**:
    - Subscribe to data updates (e.g., new pastes).
    - No need for constant polling by the client.
- **Security Concerns**:
    - **Cross-Site WebSocket Hijacking (CSWSH)**
    - **Man-in-the-Middle (MITM) Attacks** without TLS
- **DVGA Implementation**:
    - **Subscription URL**: `ws://localhost:5013/subscriptions`
    - **Mutation to Trigger Update**:
        
        ```graphql
        mutation {
          createPaste(title: "New paste", content: "Test", public: false) {
            paste {
              id
              title
              content
            }
          }
        }
        ```
        

### `Operation Names`

- **Purpose**: Uniquely identify operations in GraphQL documents.
- **Usage Context**:
    - Used when multiple operations are in a single document.
    - Optional if a single query without variables or directives.
- **Example Operations**:
    - **Single Operation (Normal Form)**:
        
        ```graphql
        query {
          pastes {
            title
          }
        }
        ```
        
    - **Single Operation (Shorthand)**:
        
        ```graphql
        {
          pastes {
            title
          }
        }
        ```
        
    - **Multiple Operations**:
        
        ```graphql
        query getPasteTitles {
          pastes {
            title
          }
        }
        query getPasteContent {
          pastes {
            content
          }
        }
        ```
        
- **Client-Defined Names**:
    - Arbitrary and potentially misleading.
    - Can disguise malicious actions (e.g., deleting data instead of retrieving).
- **Injection Risk**:
    - Operation names could be used for injection attacks if not sanitized.
    - Special characters in operation names can cause issues in logs and third-party systems.
- **Increased Attack Surface**:
    - Freedom in constructing queries increases the number of possible requests.
    - Can lead to unexpected errors in parsers not handling all query forms.

### `Fields`

- **Definition**: A field is a single piece of information within an operation's selection set, represented between `{}`.
- **Top-Level Fields**: Fields at the root level of the query.
    
    ```graphql
    {
      id
      title
      content
    }
    ```
    
- **Nested Fields**: Fields can contain their own selection sets.
    
    ```graphql
    {
      id
      title
      content
      owner {
        name
      }
    }
    ```
    
- **Security Considerations**:
    - Circular field relationships can lead to recursive, **expensive requests** that degrade performance or crash the server.
- **Field Suggestions**:
    - Servers implementing field suggestions help identify misspelled fields.
    - Example:
        
        ```
        "Cannot query field \\"titl\\" on type \\"PasteObject\\". Did you mean \\"title\\"?"
        ```
        
- **Information Disclosure**:
    - Field suggestions can be exploited to discover available fields, aiding in reconnaissance.

### `Arguments`

- **Definition**: Arguments tailor the results returned by a query. They can be applied to fields and directives.
- **Usage**:
    - Example for a top-level field:
        
        ```graphql
        query {
          users(id: 1) {
            id
            username
          }
        }
        ```
        
        - **Response**:
            
            ```json
            {
              "data": {
                "users": [
                  {
                    "id": "1",
                    "username": "admin"
                  }
                ]
              }
            }
            ```
            
    - Example for a nested field:
        
        ```graphql
        query {
          users(id: 1) {
            username(capitalize: true)
          }
        }
        ```
        
        - **Response**:
            
            ```json
            {
              "data": {
                "users": [
                  {
                    "username": "Admin"
                  }
                ]
              }
            }
            ```
            
- **Order Independence**: Arguments can be unordered without changing the query logic.
    
    ```graphql
    query {
      pastes(limit: 1, public: true) {
        id
      }
    }
    ```
    
- **Validation and Security**:
    - Arguments must match expected types (e.g., string vs. integer).
    - Applications must validate arguments beyond GraphQL's type system to prevent vulnerabilities.
    - Custom scalar types from libraries (e.g., `graphql-scalars`) can help with validation but may also introduce vulnerabilities.
    - Example of validation bypass vulnerability (CVE-2021-29921 in Python's `ipaddress` library).
- **Injection Risks**: Client-driven argument values can be vectors for injection attacks if not properly sanitized.

### `Aliases`

- **Purpose**: Aliases enable clients to **rename** field response keys to something different from the original field name.
- **Usage Example**:
    
    ```graphql
    query {
      pastes {
        myalias: title
      }
    }
    ```
    
    - **Response**:
        
        ```json
        {
          "data": {
            "pastes": [
              {
                "myalias": "My Title!"
              }
            ]
          }
        }
        ```
        
- **Handling Key Conflicts**: Aliases resolve conflicts when querying the same field with different arguments.
    - Example with conflicts and resolution using aliases:
        
        ```graphql
        query {
          queryOne: pastes(public: false) {
            title
          }
          queryTwo: pastes(public: true) {
            title
          }
        }
        ```
        
        - **Response**:
            
            ```json
            {
              "data": {
                "queryOne": [
                  {
                    "title": "My Title!"
                  }
                ],
                "queryTwo": [
                  {
                    "title": "Testing Testing"
                  }
                ]
              }
            }
            ```
            
- **Limitations**: Aliases generally support **alphanumeric characters**; special characters may cause syntax errors in GraphQL servers.
- **Security Implications**: Aliases can be used maliciously for DoS attacks (Chapter 5) and to bypass authentication controls (Chapter 7).

### `Fragments`

- **Purpose**: Fragments allow clients to define **reusable sets** of fields in GraphQL queries to enhance readability and reduce redundancy.
- **Definition**:
    
    ```graphql
    fragment CommonFields on PasteObject {
      title
      content
    }
    ```
    
    - Here, `CommonFields` is a fragment defined for `PasteObject` type, encompassing fields like `title` and `content`.
- **Usage Example**:
    
    ```graphql
    query {
      pastes {
        **...**CommonFields
      }
    }
    ```
    
    - **Result**: The fields defined in `CommonFields` (i.e., `title` and `content`) will be included wherever `...CommonFields` is used in the query.
- **Benefits**: Fragments enhance maintainability by centralizing field selections and promoting code reuse across queries.
- **Limitations**:
    - Fragments cannot accept arguments in their current GraphQL specification.
    - **Circular references** among fragments can lead to performance issues and potential Denial-of-Service (DoS) vulnerabilities.
- **Future Considerations**: Ongoing discussions within the GraphQL Working Groups may introduce fragment arguments, enhancing their flexibility and use cases.

### `Variables`

- **Purpose**: Variables allow dynamic values to be passed to GraphQL operations, enhancing flexibility and efficiency by avoiding runtime string building.
- **Definition**:
    
    ```graphql
    query publicPastes($status: Boolean!){
      pastes(public: $status){
        id
        title
        content
      }
    }
    ```
    
    - Defines a variable `$status` of type `Boolean!` (required boolean value) for the `public` argument of the `pastes` query.
- **Usage Example**:
    - Variables can be set with values like `false` or `true`, avoiding hardcoded values directly in the query.
- **Altair Example**:
    - In Altair, variables are managed in the Variables pane, allowing easy configuration of variable names and values for queries.
- **Benefits**:
    - Enhances query reusability and readability by separating dynamic values from the static query structure.
    - Facilitates easier integration and maintenance of GraphQL queries in applications.
- **Implementation**:
    - Variables are declared at the beginning of the GraphQL operation and can be passed with default values or dynamically through client applications.

### `Directives`

- **Purpose**: Directives modify the behavior of fields in GraphQL queries, allowing conditional inclusion, deprecation signaling, and custom behavior enhancements.
- **Types of Directives**:
    - **Query-Level Directives**: Used within query documents to conditionally include or skip fields.
        - **@skip**: Omits a field from the response based on a condition.
        - **@include**: Conditionally includes a field in the response based on a condition.
    - **Schema-Level Directives**: Used within schema definitions to signal deprecation or provide custom functionality.
        - **@deprecated**: Marks a field or type as deprecated, providing a reason and guiding clients away from its usage.
        - **@specifiedBy**: Specifies a URL for a custom scalar type's specification.
        - **Custom Directives**: Developed by GraphQL API developers to extend functionality beyond standard directives.
- **Usage Examples**:
    - **@skip Example**:
        
        ```graphql
        query pasteDetails($pasteOnly: Boolean!) {
          pastes {
            id
            title
            content
            owner @skip(if: $pasteOnly) {
              name
            }
          }
        }
        ```
        
        - Skips the `owner` field and its nested fields if `$pasteOnly` is `true`.
    - **@deprecated Example**:
        
        ```graphql
        type PasteObject {
          ipAddr: String @deprecated(reason: "We no longer log IP addresses")
        }
        ```
        
        - Marks `ipAddr` field as deprecated with a specified reason.
    - **Custom Directives Example**:
        
        ```graphql
        type User {
          firstName: String
          lastName: String
          fullName: String @computed(value: "$firstName $lastName")
        }
        ```
        
        - Uses a custom `@computed` directive to compute `fullName` from `firstName` and `lastName`.
- **Impact and Considerations**:
    - Directives provide powerful capabilities but can introduce vulnerabilities if not implemented securely.
    - Custom directives can expand GraphQL functionality but require careful design and testing to mitigate risks.

## `Data Types`

### `Objects`

- **Custom Object Types**: Groups of fields defining domain- or **application-specific** objects.
- **Example**: `PasteObject` type in DVGA’s schema.
    - **Fields**:
        - `id: ID!` (required, non-null)
        - `title: String`
        - `content: String`
        - `public: Boolean`
        - `userAgent: String`
        - `ipAddr: String`
        - `ownerId: Int`
        - `burn: Boolean`
        - `owner: OwnerObject` (custom object type)
- **Non-Null Wrapping Types**: `!` indicates required fields.
- **Relationships**: One-way link between `Paste` and `Owner` objects.
- **Practical Use**: Requesting an `Owner` object and its fields through a `Paste` object.

### `Scalars`

- **Core Built-in Scalars**:
    - Types: `ID`, `Int`, `Float`, `String`, `Boolean`
    - Characteristics: No fields of their own.
- **Custom Scalars**:
    - Defined by implementations.
    - Example: `DateTime` scalar in DVGA’s `PasteObject`.
- **Usage in Object Types**:
    - Example: `createdAt: DateTime!` in `PasteObject`.
- **Functionality of Custom Scalars**:
    - Ensures proper serialization, formatting, and validation.
- **Specification URL**:
    - Use `@specifiedBy` directive.
    - Example: `scalar UUID @specifiedBy(url: "https://tools.ietf.org/html/rfc4122")`

### `Enums`

- **Enums (Enumeration Types)**:
    - Purpose: Return a single string value from a list of possible values.
    - Example: `UserSortEnum` for sorting types (e.g., by username, email, password, or date joined).
- **Schema Example**:
    - Define `UserSortEnum` with fields: `ID`, `EMAIL`, `USERNAME`, `DATE_JOINED`.
    - Use `UserSortEnum` in an input type `UserOrderType` with a `sort` field.
    - Expose a `users` query with arguments `limit` and `order` (`UserOrderType`).
- **Practical Use**:
    - Example Query:
        
        ```graphql
        query {
          users(limit: 100, order: {sort: ID})
        }
        ```
        
- **Security Considerations**:
    - Risk: Allowing clients to sort by ID might expose the identity of high-value accounts (e.g., super-admin).

Feel free to provide more content for further summarization!

### `Unions`

- **Union Types**:
    - Purpose: Return one of many object types.
    - Usage: Allows a single request to get a list of different objects.
- **Example**:
    - Query using `search` feature in DVGA for both `User` and `Paste` objects:
        
        ```graphql
        query {
          search(keyword: "p") {
            ... on UserObject {
              username
            }
            ... on PasteObject {
              title
              content
            }
          }
        }
        ```
        
- **Response Example**:
    - Matches objects containing the keyword:
        
        ```json
        {
          "data": {
            "search": [
              {
                "title": "This is my first paste",
                "content": "What does your room look like?"
              },
              {
                "id": "2",
                "username": "operator"
              }
            ]
          }
        }
        ```
        
- **Schema Definition**:
    - Define a union `SearchResults`:
        
        ```graphql
        union SearchResults = UserObject | PasteObject
        ```
        
    - Define object types:
        
        ```graphql
        type UserObject {
          id: ID!
          username: String!
        }
        type PasteObject {
          id: ID!
          title: String
          content: String
        }
        ```
        
    - Define query:
        
        ```graphql
        type Query {
          search(keyword: String): [SearchResults!]
        }
        ```
        

### `Interfaces`

- **Interfaces**:
    - Purpose: Define a list of fields that **must be included** across all implementing object types.
    - Difference from Unions: Requires the **same fields** to be present in all implementing objects.
- **Example**:
    - Define an interface `SearchItem`:
        
        ```graphql
        interface SearchItem {
          keywords: [String!]
        }
        ```
        
    - Implement interface in object types:
        
        ```graphql
        type UserObject implements SearchItem {
          id: ID!
          username: String!
          keywords: [String!]
        }
        type PasteObject implements SearchItem {
          id: ID!
          title: String
          content: String
          keywords: [String!]
        }
        ```
        
    - Define a query using the interface:
        
        ```graphql
        type Query {
          search(keyword: String): [SearchItem!]!
        }
        ```
        
- **Practical Use**:
    - Allows a search query to retrieve `User` and `Paste` objects using the `keywords` field.
    - Example Query:
        
        ```graphql
        query {
          search(keyword: "example") {
            keywords
          }
        }
        ```
        
- **Security Considerations**:
    - Risk: Interfaces can expose sensitive fields if not properly decorated with authorization directives.
    - Challenge: Large SDL files may lead to missed authorization directives, posing a security risk.

### `Inputs`

- **Purpose**:
    - Handle large and complex inputs for server requests.
    - Organize client requests and reuse inputs in multiple arguments.
- **Characteristics**:
    - Similar to object types but used only as inputs for arguments.
    - Improve API structure and schema documentation readability.
- **Example**:
    - Define an input type and use it in a mutation:
        
        ```graphql
        mutation newUser($input: UserInput!) {
          createUser(userData: $input) {
            user {
              username
            }
          }
        }
        ```
        
    - JSON object for input:
        
        ```json
        {
          "input": {
            "username": "tom",
            "password": "secret",
            "email": "tom@example.com"
          }
        }
        ```
        
    - Input type in schema:
        
        ```graphql
        input UserInput {
          username: String!
          password: String!
          email: String!
        }
        ```
        
- **Usage**:
    - Example mutation request with input type:
        
        ```graphql
        mutation {
          newUser(input: {
            username: "tom",
            password: "secret",
            email: "tom@example.com"
          }) {
            user {
              username
            }
          }
        }
        ```
        
- **Security Considerations**:
    - Potential risk: Clients might bypass type validations if there are validation logic flaws.
    - Example: Issues with custom scalar validations (e.g., IP addresses, email addresses) can lead to bypassing registration or performing injections.

## `Introspection`

- **Power of Introspection**:
    - Enables clients to discover actions they can take using a GraphQL API.
    - Allows querying for schema details: queries, mutations, subscriptions, directives, types, fields, etc.
    - Valuable for reconnaissance, profiling, data collection, and attack-vector analysis.
- **Introspection Types**:
    - `__Schema`: Information about the schema of a GraphQL service.
    - `__Type`: Information about a type.
    - `__TypeKind`: Different kinds of types (scalars, objects, interface, union, enum, etc.).
    - `__Field`: Information for each field of an object or interface type.
    - `__InputValue`: Field and directive argument information.
    - `__EnumValue`: Possible values of an enum.
    - `__Directive`: Information on custom and built-in directives.
- **Example Introspection Query**:
    - Query for schema types:
        
        ```graphql
        query {
          __schema {
            types {
              name
            }
          }
        }
        ```
        
    - Response example:
        
        ```json
        {
          "data": {
            "__schema": {
              "types": [
                { "name": "PasteObject" },
                { "name": "ID" },
                { "name": "String" },
                { "name": "OwnerObject" },
                { "name": "UserObject" }
              ]
            }
          }
        }
        ```
        
- **Detailed Type Investigation**:
    - Query for fields within `PasteObject`:
        
        ```graphql
        query {
          __type(name: "PasteObject") {
            name
            kind
            fields {
              name
              type {
                name
                kind
              }
            }
          }
        }
        ```
        
    - Response example:
        
        ```json
        {
          "__type": {
            "name": "PasteObject",
            "kind": "OBJECT",
            "fields": [
              { "name": "id", "type": { "name": null, "kind": "NON_NULL" } },
              { "name": "title", "type": { "name": "String", "kind": "SCALAR" } },
              { "name": "content", "type": { "name": "String", "kind": "SCALAR" } },
              { "name": "owner", "type": { "name": "OwnerObject", "kind": "OBJECT" } }
            ]
          }
        }
        ```
        
- **Security Considerations**:
    - Sensitive fields can be exposed if included in the schema with introspection enabled.
    - Introspection provides extensive API intelligence, akin to a REST API Swagger (OpenAPI) definition.
    - Enables discovery of queries, mutations, and subscriptions, their arguments, and crafting of operations.
