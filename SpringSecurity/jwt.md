# Chapter 1: Introduction

JSON Web Token, or JWT, is an open standard that defines a compact and secure way of transmitting information between two parties.

To understand JWTs, it is important to first understand session tokens.

# Chapter 2: Session Tokens

Session tokens are encrypted strings that are used to identify a session between two parties.

Here is an example of how session tokens work.

Imagine you are calling customer service. You tell the representative your problem, and they take some notes. At the end of the call, they give you a case number. The case number is a session token that identifies your conversation with the representative.

The next time you call customer service, you can provide the same case number. This will allow your representative to quickly look up your conversation and continue helping you.

Now let's think the same way with clients and servers.

Imagine you want to create an account on a website. You fill out a registration form and click Submit. The website sends you a token that is stored in browser cookies. This token allows you to access the website and its features.

Session tokens are a good way to keep track of your users and their sessions. However, they have one big drawback: they can only be used on the server that issued them.

For example, let's say you log into your bank's website. The bank server will issue a token that allows you to access the website. However, if you try to access the bank's mobile app, the app won't be able to verify your token because it was issued by a different server.

This is where JSON Web Tokens come in. JWTs are a type of token that can be used on multiple servers.

# Chapter 3: JSON Web Tokens

JWTs can be used on multiple servers, and this makes them ideal for applications that need to authenticate users across multiple systems.

Imagine you are going to a concert. When you buy your ticket, you are given a wristband. The wristband is a way for the security guards to quickly identify you as a concertgoer.

JWT tokens are like wristbands for the internet. They are a way for servers to quickly identify users and their permissions. This makes it easier for users to access the resources they need, and it makes it more secure for servers to protect their data.

JWTs are a way for securely transmitting information between two parties, such as an authentication token or a user's profile information.

For example, ` let's say you have a front-end  application that communicates with an API. You can use a JWT to authorize the user and check their role before allowing them to access the API. This can help you protect your API from unauthorized access.`

To do this securely, `the server will issue the user a JWT that contains the username, role, and random token.`

`The user's browser will then store the token in local storage. When the user makes a request to an API, they will include the token in the request header.`

`The API will then verify the token and check the user's role.`

If the user is authorized to access the API, the API will respond with the requested data. Otherwise, the API will return an error.

This method of authorization is very efficient because it only requires the user to authenticate once. The user can then access any API that they are authorized to access without having to re-authenticate each time.

JWTs are also very useful in distributed systems and microservices architecture.

# Chapter 4: Distributed Systems

In this type of architecture, there are many different services that need to communicate with each other. JWTs can be used to securely authenticate users and services and to perform role checks.

This can help to ensure that only authorized users and services can access each other.

The private/public key signing method is a very secure way to sign JWTs. ` It uses a private key to sign the JWT and a public key to verify the JWT.`

This method is very secure because it is very difficult to forge a JWT that is signed with a private key.

Now, using JWTs with the private/public key signing method can save you a huge amount of requests and improve the overall scalability of your application.

This is because the user only needs to authenticate once, and the JWT can be used to access multiple services.

This can significantly reduce the load on your authentication server.

So, if you have a special envelope called a JWT, inside this envelope there are three parts: a return address, a message, and a seal.

# Chapter 5: JWT

The return address is like the header on the outside of the envelope. It tells the recipient who sent the JWT and where it came from.

It is usually written in a format called JSON, which is just a way to organize information using curly braces and colons.

The header contains metadata information about the JSON Web Token.

The algorithm here is used to sign the token, and this is useful for the attempted reproduction of the signature, which we'll talk about later.

The type of the token, in the case of a JWT, will always have the `JWT` value.

You will sometimes find extra headers that were added by the sender or issuer, but the above two will almost always be there.

The message is like the actual content or payload of the JWT.

The payload contains the claims, which are statements about the subject of the JWT, such as the user's identity, their permissions, or the time at which the JWT was issued.

There are `two types of JWT claims.`

`Registered claims` are the claims that are defined by the JWT specification.

Some examples of registered claims include:

* Issuer of the JWT
* Subject of the JWT
* Audience of the JWT
* Expiration time
* Issue time of the JWT

`Custom claims `are claims that are not defined by the JWT specification but can be used to store any type of information that you want to include in the JWT.

The claims in the JWT are encoded in JSON and are separated by dots.

For example, a JWT with the following claims:

`iss` is the issuer. The issuer is the entity that generated and issued the JWT. This could be your company, your website, or any other entity that you want to identify yourself as.

`sub`, or subject, is the entity that is identified by the JWT. This could be a user, a device, or any other entity that you want to authenticate. This is the subject of your token. This is the entity that is identified by the token.

`aud`, or audience, is the target audience of the JWT. This could be a specific group of users, such as your beta testers, or it could be a public audience.

The audience for the token is `/api/my-api`, and this is the entity that the token is intended for.

`exp`, or expiry, is the timestamp after which the JWT should not be accepted. This helps to prevent the JWT from being used after it is expired.

`iat`, or issued at, is the date at which the JWT was issued and helps to track the age of the JWT and prevent it from being used before it was issued.

Now you can use custom claims that you need to store data in your JWTs, such as roles. You could store the user's role, their permissions, or any other data that you need to use to authenticate or authorize the user.

The payload of a JSON Web Token is, by default, decoded by anyone. In fact, you can paste any JWT into JWT.io and immediately see the claims.

Now the seal is what makes the JWTs secure. It is like a special signature that only the sender knows how to create.

The seal is created using a secret code, kind of like a password.

The seal ensures that nobody can tamper with the contents of the JWT without the sender knowing about it.

The signature is created from the encoded header, encoded payload, a secret, which can be a private key, and a cryptographic algorithm.

# Chapter 6: Code

All these components allow the creation of the signature.

The following code shows how to generate the JWT and the signature.

The code first imports the JWT library, and this library is used to generate and decode JWTs.

We then define two variables: `header` and `payload`.

The header variable contains the header of the JWT, which includes the algorithm used to sign the JWT and the type of the JWT.

The payload variable contains the payload of the JWT, which includes the claims of the JWT.

We then use the JWT encode function to encode the header and payload using the secret and the HMAC SHA-256 algorithm.

The JWT encode function returns two values: the encoded header and the encoded payload.

The code then uses the HMAC module to calculate the signature of the JWT.

The signature is calculated using the HMAC SHA-256 algorithm and the secret.

We then use the encoded header, the encoded payload, and the signature to form the JWT.

The JWT is a string that is separated by periods, and finally we print the JWT.

The signature is probably the most important and misunderstood part of JWTs.

Here is a sample signature. The signature looks like gibberish, but this is a unique and reproducible value.

# Chapter 7: Signature

JSON Web Tokens are decodable by anyone.

In fact, feel free to copy the following token and paste it directly into JWT.io. You can immediately see all the claims in this token.

This is why you should never store sensitive information in the token.

And by the way, a user's role is not sensitive, but a password is.

In a real-world application, it would be better to store the user's username and a role in the JWT and store the password in a secure database.

The server would then use the user's username and password to authenticate the user and issue them a JWT with a username, role, and a random token.

The user would then store the token in the browser and use it to authenticate with the API.

Now you might be wondering, how can we prevent people from tampering with a token?

The answer is the signature.

The seal in our envelope is like the signature on a JWT. It proves that the package has not been tampered with since you sent it.

It proves that the JWT has not been tampered with since it was issued.

So, if somebody tampered with the claims and set their role from `user` to `admin`, the JWT verification will fail because the signature does not match anymore.

The signature is generated using the original payload defined by the issuer.

This makes JWTs a secure way to transfer information between two parties.

Remember, the purpose of JWT is not to hide data but to ensure the authenticity of the data.

JWT is signed and encoded, and not encrypted.

JWTs are a versatile tool that can be used for a variety of purposes. They are a secure and efficient way to transmit information between two parties.

