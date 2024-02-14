# qaul-proposal

# About you

| Name | Mohit Kumar |
| --- | --- |
| Email | madscientists1523@gmail.com |
| Blog | https://claddy.hashnode.dev |
| Github | https://github.com/claddyy |
| Phone | +91 7701986595 |
| Country/Region | India |

# Your biography

### `Who are you? What's the focus of your recent work? What qualifies you for the project idea?`

I’m Mohit Kumar a third-year undergraduate at the Indian Institute of Information Technology, Nagpur, India. I respect and love freedom tech(decentralization). I’ve been learning rust, and focusing on operating systems for over 3 months. The inspiration for learning Rust came when I started to go deep down in Bitcoin Foss, and I realized most of the software infrastructure is written in Rust there. I’m also a full-stack developer(MERN stack). Since I’ve already been working on backend projects I know authentication and its systems. I also have experience writing backend code in Rust. However, I consider my most valuable skill to be my unwavering drive to research and learn about topics that fuel my passion.

### `Experiences in Free Open Source Software`

I’ve been using free open source software for a long time, and contributing to FOSS for a year. I have made open-source contributions in eclair and Swasth-alliance. details of my work are explained below in the next subheading.

### `Please describe your contributions to free and open-source projects.`

### eclair

`eclair` is a scala implementation of the lightning network. The Lightning Network is a **second layer for Bitcoin (BTC)** that uses micropayment channels to scale the blockchain’s capability and handle transactions more efficiently and more cheaply. It is a technological solution designed to solve glitches associated with Bitcoin by introducing off-chain transactions.

I developed a CLI tool for the lightning node operators, whose link is [https://github.com/ACINQ/eclair-cli](https://github.com/ACINQ/eclair-cli)

[https://github.com/ACINQ/eclair-cli](https://github.com/ACINQ/eclair-cli)

I raised 10+ pull requests, found here [https://github.com/ACINQ/eclair-cli/pulls](https://github.com/ACINQ/eclair-cli/pulls)

The most important of these PRs which created a structure for the whole tools are

| https://github.com/ACINQ/eclair-cli/pull/10 | Co-authored by me setting the architecture |
| --- | --- |
| https://github.com/ACINQ/eclair-cli/pull/12 | Initial commands implementation |
| https://github.com/ACINQ/eclair-cli/pull/13 | Initial commands implementation |
| https://github.com/ACINQ/eclair-cli/pull/15 | Add auto-completion for bash shell |

### Health Claims Exchange

The Health Claim Exchange(HCX) Specification is a communication protocol that facilitates the exchange of health claim information between payers, providers, beneficiaries, and other relevant entities. I developed a `JavaScript` SDK that facilitates easier integration with HCX protocol. 

[https://github.com/Swasth-Digital-Health-Foundation/integration-sdks/pull/67](https://github.com/Swasth-Digital-Health-Foundation/integration-sdks/pull/67)

The `npm` package of the same can be found at [https://www.npmjs.com/package/hcx-integrator-sdk](https://www.npmjs.com/package/hcx-integrator-sdk)

# Your GSoC Project

- Project Title: **qaul RPC user authentication layer**
- Mentor: [MathJud](http://matrix.org)
- Description:   qaul is an internet-independent communication app for p2p. Devices(nodes) connect on a local network such as LAN, internet, or BLE. Since it uses mDNS to get IP addresses of the devices in the local network, there’s low vulnerability to any data leak outside the network. This project aims to implement the user authentication layer. Currently qaul supports only one user per node, but with this project, we plan to extend the capabilities to support multiple users per node , organised with authenticated and encrypted sessions. This project lays the foundation for a secure and scalable system that accommodates various users, and is crucial for the potential development  of a web interface.

### Benefits to community networks, who would gain from your project?

This would bring several benefits to community networks and various stakeholders.

- Users: Since we will have the ability to support multiple users per node,  community network participants can encourage more individuals to join and contribute to the network. Also, with improved scalability, the network grows organically without compromising the security or the performance.
- Developers and Contributors: The project lays the foundation for a web interface. The authentication system and session managements systems create a more extensible architecture, allowing developers to build additional features and functionalities.

### Milestones

- [ ]  define the authentication system.
- [ ]  define the encryption system.
- [ ]  define the protobuf communication structure.
- [ ]  implement the sessions (into CLI application)
- [ ]  implement the Authentication System.
- [ ]  encrypt the sessions.
- [ ]  write unit and integration tests to ensure robustness.
- [ ]  relevant user and developer documentation

## Project Details

In a peer-to-peer (P2P) application like qaul,  the best approach to implement an authentication system would include the public key cryptography, token-based sessions and the noise protocol for encryption.

### Authentication & Encryption

1. **Public Key Infrastructure(PKI) for Identity verification**
    
    Each user and node in the network has a unique pair of public/private keys. The public key serves as the user’s identity, while the private key is used for signing and encryption tasks, ensuring secure communication. We use `rand` to provide cryptographically secure random numbers for key generation. To generate and manage cryptographic we can use any other crate example `ed25519-dalek`.
    
2. **Secure Registration and Key Exchange**
    
    Upon joining the network, a user generates a new key pair. The public key is broadcasted or shared with the network using a secure channel, possibly incorporating a handshake protocol facilitated by the noise protocol. The initial key exchange processes use Diffie-Hellman key exchange mechanism, integrated within the noise protocol, to establish, private communication channels.
    
3. **Token-based Authentication for sessions**
    
    After initial key exchange, generate a session token for the user. This token is encrypted with the user’s public key ensuring that only the holder of the corresponding private key can decrypt it and use it. Also, this session token is used for subsequent authentication during the session. It can contain information like the session expiry time, the user’s public key, and a unique session identifier. Since the token-based authentication is stateless, this reduces the need for central session management. It allows flexible session management, therefore would support multiple sessions per user. 
    
    And creating custom tokens would indeed a better approach for this rather than trying to use something standard like JWTs due to obvious concerns and issues which isn’t suitable for an internet independent P2P applications. An pseudocode of this implementation is provided in a prototype here 
    
    ```rust
    use ed25519_dalek::{PublicKey, SecretKey, Signature, Signer, Verifier};
    use serde::{Serialize, Deserialize};
    use bincode;
    
    #[derive(Serialize, Deserialize)]
    struct CustomToken {
        user_id: String,
        session_id: String,
        expires_at: u64,
        // more fields
    }
    
    fn generate_token(user_id: &str, session_id: &str, secret_key: &SecretKey) -> Vec<u8> {
        let token = CustomToken {
            user_id: user_id.to_owned(),
            session_id: session_id.to_owned(),
            expires_at: /*logic*/,
        };
    
        let encoded = bincode::serialize(&token).unwrap();
        let signature = secret_key.sign(&encoded);
    
        // concatenate the encoded token and signature into one Vec<u8>
        [encoded, signature.to_bytes().as_slice()].concat()
    }
    
    fn verify_token(token_bytes: &[u8], public_key: &PublicKey) -> Result<CustomToken, &'static str> {
        let (encoded, signature) = token_bytes.split_at(token_bytes.len() - 64); // Assuming signature is always the last 64 bytes
        let signature = Signature::from_bytes(signature).unwrap();
    
        public_key.verify(encoded, &signature).map_err(|_| "Verification failed")?;
    
        bincode::deserialize(encoded).map_err(|_| "Deserialization failed")
    }
    ```
    
4. **Optional Password Support**
    
    For additional security, user can opt to protect their private keys with a password. This password will never be transmitted over the network, and would be used locally ot encrypt the private key on the user’s device.
    
    We can also implement a secure password-based key derivation function(PBKDF) to strengthen the security of password-protected keys.
    
5. **Noise Protocol for Secure Communication**
    
    The noise protocol framework would be used to encrypt all communication between nodes, including the initial key exchange, session token transmission, and any data exchange thereafter. This obviously provides a flexible and robust cryptographic primitives for ensuring the confidentiality, integrity, and the authenticity of messages.
    
    It defines several handshake patterns that negotiate the cryptographic keys between parties, and when a handshake is complete it generates a pair of symmetric keys for encrypting and decrypting the messages. It also has `forward secrecy` that ensures that session keys are not compromised even if the long-term keys are. We will use `snow`  for the implementation.
    
6. **Zero-Configuration User Experience**
    
    The process of discovery and initial connection process among nodes, is already implemented using the mDNS and other local discovery protocols.
    
    We would design the underlying systems to automatically manage key generation, exchange, and session management processes, requiring minimal inputs from the user. 
    
    We will implement caching mechanism for session tokens and user preferences.
    

For writing the tests, standard test library of rust would be used (`std::test`) 
![Image might be down](https://github.com/claddyy/qaul-ptype/blob/main/qaul%20RPC%20User%20Authentication%20Process.png)
### Protobuf Structure

The basic structure of the [protobuf communication structure](https://github.com/qaul/qaul.net/blob/main/rust/libqaul/src/rpc/qaul_rpc.proto) for the project, already provides us a foundation for modular and extensible Remote Procedure Call interactions between the GUI and libqaul. We extend this to accommodate the authentication and session management functionalities. 

First, we would be updating modules enum

```rust
enum Modules {
    // Existing modules...
    AUTHENTICATION = 15;  // Adding an authentication module
    SESSION = 16;         // Adding a session management module
}
```

Next we define the Authentication and Session Management Messages. An example for this is shown below, but this isn’t finalized. 

```rust
message LoginRequest {
    string username = 1;
    bytes password = 2;  

message LoginResponse {
    string status = 1;   // "success", "failure", etc.
    bytes token = 2;     // Session token for authenticated operations
    string message = 3;  // Error messages
}
```

Similarly, we would be defining the other `session management` and `services` definitions. The protobuf definitions would also include detailed error handling messages.

## Project Size

This is a large sized project. The estimated time for this is 350 hours.

## Timeline

| Timeline | Task  | Summary |
| --- | --- | --- |
| Week 1-2 | Project Setup and Initial Research | Setup dev environment, finalize the rust crates for cryptography. Conduct a thorough review of existing http://qaul.net existing architecture and codebase. |
| Week 3-4 | Authentication, Encryption design | Define the authentication system architecture. Design the custom token generation and verification mechanism. Outline the secure registration and key exchange processes. |
| Week 5-6 | Protobuf Communication Structure | Define and extend the protobuf communication structure for authentication and session management. Implement protobuf messages and services in Rust. |
| Week 7-9 | Authentication System Implementation | Develop the authentication mechanisms based on the design document. Implement user registration, login, and logout functionalities. |
| Week 10-12 | Implementing Session Management | Develop the session management system, including session creation, validation, and expiration. Integrate session management with the authentication system. |
| Week 13-16 | Encryption of sessions | Implement encryption mechanisms for session data using the Noise protocol. Ensure secure transmission of session tokens and data. |
| Week 17-18 | CLI Application Integration | Integrate the authentication and session management systems into the qaul.net CLI application. Develop CLI commands for user authentication and session management. |
| Week 18-19 | Testing and bug fixes | Conduct thorough testing of the authentication and session management systems. Identify and fix bugs. Optimize performance based on test results. |
| Week 20-21 | Documentation and User Feedback | Create comprehensive developer and user documentation for the new systems. Release a beta version to a limited user base for feedback. |
| Week 22 | Final testing, and refinement | Refine the systems based on user feedback. Conduct final testing of all systems. |

## Availability

| Now - 1 May | available | 9am-5pm CEST |
| --- | --- | --- |
| 1 May - 20 May | exams | not available due to university exams |
| 20 May - 20 July | available | 9am-5pm CEST |
| 20 July - 5 November | partially available | 1pm-9pm CEST |

*Available*: I’m very flexible with the timings, whenever I’m available. Number of hours, won’t be an issue. The timings of the weekly meetings and other synchronous activities can be scheduled as per your convenience.

# After GSoC

Even after the completion of the Google Summer of Code program, I plan to remain actively involved in the project, dedicating my free time to contribute in meaningful ways. This will involve submitting pull requests and conducting code reviews, all to enhance the project's functionality and impact. Additionally, I am keen to explore other intriguing ideas listed in [https://projects.freifunk.net/#/projects](https://projects.freifunk.net/#/projects), should they not have been taken by other students.
