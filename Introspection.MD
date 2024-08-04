- Code:

```graphql
{
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    subscriptionType {
      name
    }
    types {
      ...FullType
    }
    directives {
      name
      description
      locations
      args {
        ...InputValue
      }
    }
  }
}

fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
    isDeprecated
    deprecationReason
  }
  inputFields {
    ...InputValue
  }
  interfaces {
    ...TypeRef
  }
  enumValues(includeDeprecated: true) {
    name
    description
    isDeprecated
    deprecationReason
  }
  possibleTypes {
    ...TypeRef
  }
}

fragment InputValue on __InputValue {
  name
  description
  type {
    ...TypeRef
  }
  defaultValue
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
                ofType {
                  kind
                  name
                }
              }
            }
          }
        }
      }
    }
  }
}

```

- [ ]  If introspection is disabled, use [clairvoyance](https://github.com/nikitastupin/clairvoyance)
    - **Development Subdomains**:
        - Often, GraphQL introspection is enabled on development, staging, or UAT subdomains where strict security measures might not be fully implemented. These environments can be easier targets for schema enumeration.
    - **Clairvoyancex Tool**:
        - **Usage**: Use custom wordlists and proxy through tools like Burp Suite to intercept and modify requests. Example command:
            
            ```bash
            python3 -m clairvoyancex -vv -o schema411.json -w google-10000-english.txt https://example.com/api/graphql -t 100
            ```
            
            - `vv`: Verbose mode for detailed output.
            - `o schema411.json`: Output file to save the enumerated schema in JSON format.
            - `w google-10000-english.txt`: Custom wordlist to use for query guessing.
            - `t 100`: Timeout set to 100 seconds per request to manage response times.

- **JS/App Analysis**:
    - **Reverse Engineering**: Analyze JavaScript code or mobile app binaries to understand how queries and mutations are structured and used. This approach can reveal GraphQL endpoints, parameters, and potential vulnerabilities.
- **Basic Security Testing**:
    - **Explore Bug Classes**: Apply common security testing techniques such as IDOR (Insecure Direct Object Reference), SQL Injection (SQLi), File Disclosure, SSRF (Server-Side Request Forgery), and RCE (Remote Code Execution) specifically adapted for GraphQL endpoints. These vulnerabilities can sometimes be discovered even without explicit enumeration.
