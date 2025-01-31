# Component Description

## Storage Provider
MySQL Database with JPA interface implementation. The tables are well designed with multiple constraints and indexes. Thus, we can guranatee the data consistency.

## External data sources
The component is able to connect to the following external data sources:
- Mapbox
- Local cache

## User info controller
This component can be used to manage user information, i.e. update email addresses, passwords, bio, etc. And it also can be used to manage user's rating and journey histories.

## Journey controller
This component can be used to manage user's journey. Users can create a new journey, apply to join a journey, etc. Also, a journey host can allow other users to join their journey and start a joureny. Other users can confirm arrived in this component. A journey with all users arrived will be marked as finished.

## User review controller
Once a journey is finished, users can leave a review for other users. The rating is from 1 to 5 with text description. Users can check others' information including reviews in a journey searching result page.

## Message controller
This component can be used to manage user's message. Users can send a message to other users, i.e. any person in a journey, before the user joins the journey, to make sure every details are clear. The message controller is peer-to-peer based. Users have a public key and a private key. The public key is used to send messages to other users. The private key is used to decrypt messages from other users. Both public and private keys are stored in the database, but the private key is encrypted with user's password. And user's password in the database is hashed, so the server cannot know the user's password in plain text.

## History controller
This component can be used to manage user's history. Users can check their journey history, including journey's name, start time, end time, from where to where, etc.

## Map service
The component is based on Mapbox's API. The component can be used to search for a place, get the place's information, and get the place's map, etc.

## Local cache service
This service is used to enhance the user experience. For example, we don't have to search for a place every time when users search for a place. We can cache the search result and use it when users search for a place.

## Web application
Our frontend is based on React.js, and it is a single page application(SPA). With SPA, users don't need to jump between different pages which is good for user experience.

## Token authentication controller
The controller is used to authenticate users. The token is generated by the server and sent to the client with HTTPS protocol. The client can use the token to access the server. Also it providers a interface to refresh the token, check the token is valid, etc.

## Login/register controller
This component is used to login and register. After logged in, the server will generate a token and send it to the client. The client can use the token to access the server.

# Component APIs

## Register/Login controller

### Register
User need to provide a username, password, email, and bio. The username and password are used to login. Public key and private key are used to finish peer-to-peer communication.

```typescript
register(request: RegisterRequest): RegisterResponse

interface RegisterRequest {
  username: string
  password: string
  publicKey: string
  privateKey: string
  timestamp: number
}

interface RegisterResponse {
  success: boolean
  reason: string
}
```

### Login
Give a username and a password to login. The server will check the username and password. If the username and password are correct, the server will generate a token and send it to the client. If not, the server will return the reason.

```typescript
login(request: LoginRequest): LoginResponse

interface LoginRequest {
  username: string
  password: string
  timestamp: number
}

interface LoginResponse {
  success: boolean
  reason: string
  token: string
}
```

## User info controller

### Get Profile
To get a user's profile, the user need to provide a token and a user id. The server will check the token is valid. If the token is valid, the server will return the user's profile. If not, the server will return the reason.

```typescript
getProfile(request: GetProfileRequest): GetProfileResponse

interface GetProfileRequest {
  token: string
  timestamp: number
  payload: {
    userId: string
  }
}

interface GetProfileResponse {
  success: boolean
  reason: string
  payload: {
    userId: string
    username: string
    email: string
    bio: string
    rating: number
    journeyHistory: {
      [journeyId: string]: {
        name: string
        startTime: string
        endTime: string
        from: string
        to: string
        isFinished: boolean
      }
    }
  }
}
```

### Get Avatar
This API is used to get a user's avatar.

```typescript
getAvatar(request: GetAvatarRequest): btyes[]

interface GetAvatarRequest {
  token: string
  timestamp: number
  payload: {
    userId: string
  }
}
```

### Update User Info
This API is used to update a user's profile. The server can know the user id from the token.

```typescript
updateUserInfo(request: UpdateUserInfoRequest): UpdateUserInfoResponse

interface UpdateUserInfoRequest {
  token: string
  timestamp: number
  payload: {
    bio: string
    avatar: string
  }
}

interface UpdateUserInfoResponse {
  success: boolean
  reason: string
  payload: {
    userInfo: {
      userId: string
      username: string
      email: string
      bio: string
    }
  }
}
```

### Get History
This API is used to get a user's history. The server can know the user id from the token.

```typescript
getHistory(request: GetHistoryRequest): GetHistoryResponse

interface GetHistoryRequest {
  token: string
  timestamp: number
  payload: {
    from: number
    len: number
  }
}

interface GetHistoryResponse {
  success: boolean
  reason: string
  payload: {
    history: {
      [journeyId: string]: {
        name: string
        startTime: string
        endTime: string
        from: string
        to: string
        isFinished: boolean
      }
    }
  }
}
```

### Get Review
This API is used to get a user's review. The server can know the user id from the token.

```typescript
getReview(request: GetReviewRequest): GetReviewResponse

interface GetReviewRequest {
  token: string
  timestamp: number
  payload: {
    from: number
    len: number
  }
}

interface GetReviewResponse {
  success: boolean
  reason: string
  payload: {
    review: {
      [userId: string]: {
        rating: number
        description: string
      }
    }
  }
}
```

## Journey controller

### approve join
This API is used to approve a user to join a journey. Only the host can call this API.

```typescript
approveJoin(request: ApproveJoinRequest): ApproveJoinResponse

interface ApproveJoinRequest {
  token: string
  timestamp: number
  payload: {
    userId: string
  }
}

interface ApproveJoinResponse {
  success: boolean
  reason: string
}
```

### get unapproved
This API is used to get a list of unapproved users. Only the host can call this API.

```typescript
getUnapproved(request: GetUnapprovedRequest): GetUnapprovedResponse

interface GetUnapprovedRequest {
  token: string
  timestamp: number
}

interface GetUnapprovedResponse {
  success: boolean
  reason: string
  payload: {
    unapproved: {
      userId: string[]
    }
  }
}
```

### confirm arrive
This API is used to confirm a user has arrived at the destination.

```typescript
confirmArrive(request: ConfirmArriveRequest): ConfirmArriveResponse

interface ConfirmArriveRequest {
  token: string
  timestamp: number
  payload: {
    userId: string
  }
}

interface ConfirmArriveResponse {
  success: boolean
  reason: string
}
```

### create journey
This API is used to create a journey. The user need to provide a start point and a destination. The server will generate a journey id and send it to the user.

```typescript
create(request: CreateJourneyRequest): CreateJourneyResponse

interface CreateJourneyRequest {
  token: string
  timestamp: number
  payload: {
    from: string
    to: string
  }
}

interface CreateJourneyResponse {
  success: boolean
  reason: string
  payload: {
    journeyId: string
  }
}
```

### exit journey
This API is used to exit a journey. The user need to provide a journey id. The server will check the journey id is valid. If the journey id is valid, the server will remove the user from the journey.

```typescript
exit(request: ExitJourneyRequest): ExitJourneyResponse

interface ExitJourneyRequest {
  token: string
  timestamp: number
  payload: {
    userId: string
  }
}

interface ExitJourneyResponse {
  success: boolean
  reason: string
}
```

### get by id
This API is used to get a journey by id.

```typescript
getById(request: GetByIdRequest): GetByIdResponse

interface GetByIdRequest {
  token: string
  timestamp: number
  payload: {
    journeyId: string
  }
}

interface GetByIdResponse {
  success: boolean
  reason: string
  payload: {
    journey: {
      name: string
      startTime: string
      endTime: string
      from: string
      to: string
      isFinished: boolean
      users: {
        userId: string
        username: string
        email: string
        bio: string
        rating: number
        avatar: string
      }
    }
  }
}
```

### get by location
This API is used to get a list of journeys by location.

```typescript
getByLocation(request: GetByLocationRequest): GetByLocationResponse

interface GetByLocationRequest {
  token: string
  timestamp: number
  payload: {
    from: string
    to: string
  }
}

interface GetByLocationResponse {
  success: boolean
  reason: string
  payload: {
    journeys: {
      [journeyId: string]: {
        name: string
        startTime: string
        endTime: string
        from: string
        to: string
        isFinished: boolean
        users: {
          userId: string
          username: string
          email: string
          bio: string
          rating: number
          avatar: string
        }
      }
    }
  }
}
```

### join
This API is used to join a journey. The user need to provide a journey id. The server will check the journey id is valid. If the journey id is valid, the server will add the user to the journey.

```typescript
join(request: JoinJourneyRequest): JoinJourneyResponse

interface JoinJourneyRequest {
  token: string
  timestamp: number
  payload: {
    journeyId: string
  }
}

interface JoinJourneyResponse {
  success: boolean
  reason: string
}
```

### start
This API is used to start a journey. Only the host can call this API.

```typescript
start(request: StartJourneyRequest): StartJourneyResponse

interface StartJourneyRequest {
  token: string
  timestamp: number
}

interface StartJourneyResponse {
  success: boolean
  reason: string
}
```


## User review controller

### get by user id
This API is used to get a list of reviews by user id.

```typescript
get(request: GetByUserIdRequest): GetByUserIdResponse

interface GetByUserIdRequest {
  token: string
  timestamp: number
  payload: {
    userId: string
  }
}

interface GetByUserIdResponse {
  success: boolean
  reason: string
  payload: {
    reviews: {
      [userId: string]: {
        rating: number
        description: string
      }
    }
  }
}
```

### create
This API is used to create a review. The user need to provide a rating and a description.

```typescript
createReview(request: CreateReviewRequest): CreateReviewResponse

interface CreateReviewRequest {
  token: string
  timestamp: number
  payload: {
    userId: string
    revieweeId: string
    rating: number
    description: string
  }
}

interface CreateReviewResponse {
  success: boolean
  reason: string
}
```

## Message controller

### send
This API is used to send a message.

```typescript
send(request: SendMessageRequest): SendMessageResponse

interface SendMessageRequest {
  token: string
  timestamp: number
  payload: {
    receiver: string
    content: string
  }
}

interface SendMessageResponse {
  success: boolean
  reason: string
}
```

### get
This API is used to get a list of messages.

```typescript
get(request: GetMessageRequest): GetMessageResponse

interface GetMessageRequest {
  token: string
  timestamp: number
  payload: {
    from: number
    len: number
  }
}

interface GetMessageResponse {
  success: boolean
  reason: string
  payload: {
    messages: {
      [id: string]: {
        sender: string
        receiver: string
        content: string
        timestamp: number
      }
    }
  }
}
```

### get public key
This API is used to get the public key of the user.

```typescript
getPublicKey(request: GetPublicKeyRequest): GetPublicKeyResponse

interface GetPublicKeyRequest {
  token: string
  timestamp: number
  payload: {
    userId: string
  }
}

interface GetPublicKeyResponse {
  success: boolean
  reason: string
  payload: {
    publicKey: string
  }
}
```

## Token authentication controller

### check token
This API is used to check the token is valid.

```typescript
checkToken(request: CheckTokenRequest): CheckTokenResponse

interface CheckTokenRequest {
  token: string
  timestamp: number
}

interface CheckTokenResponse {
  success: boolean
  reason: string
  payload: {
    available: boolean
  }
}
```

### refresh token
This API is used to refresh the token.

```typescript
refreshToken(request: RefreshTokenRequest): RefreshTokenResponse

interface RefreshTokenRequest {
  token: string
  timestamp: number
}

interface RefreshTokenResponse {
  success: boolean
  reason: string
  payload: {
    token: string
  }
}
```

### info
This API is used to get the information of the user by token.

```typescript
info(request: InfoRequest): InfoResponse

interface InfoRequest {
  token: string
  timestamp: number
}

interface InfoResponse {
  success: boolean
  reason: string
  payload: {
    userId: string
    username: string
    email: string
    bio: string
    rating: number
    avatar: string
  }
}
```

# Quality of Service Technical Requirements

## Security Requirements

---
- Do transactions need to be encrypted?
- Level of encryption? (e.g., 40-bit encryption in US)

Yes, they do need.
1. HTTPS with 2048-bit private key encryption
2. SHA256 for user password management.

---
User Identification
- uid/pw, cookies, certificates, application-level?
- Existing customer database that should be used to identify online visitors?


We use both uid/pw and token to identify a user. To be specific, user use uid and password to get the token, then use the token to visit services.

To get the token, we need to visit the database to compare if the password is correct. And once a user get a token, he/she don need to visit the database again.

---
Access to data
- Do you need to restrict access to parts of the site?
- What privacy rules should be applied to information provided by users

Yes, some functions are only for logged in users, e.g., journey, review, message, etc.

To protect privacy, a journey content which includes sensitive data (user locations) can be only accessed by the user in the journey.

---
What are the legal requirements and policies for auditing content, changes and transactions?


We follow GDPR

---
Do you plan to use a secure demilitarised zone into which your project server code could be placed?


Yes, the core services run in k8s with docker. This network virtualization and program containerization provide more securities than traditional systems.



---
## System Management

---
Do you have access to the infrastructure required to install and run you own server?
Yes, we have.

---
What are the response time targets?


20 – 50 ms in average

---
Availability:
- What hours should the service be available?

- Is it acceptable to have any scheduled downtime for maintenance?

- How important is it that the service be never interrupted, even for unscheduled component failures?

- If interruptions do occur, what should be the target time for resuming service?


1. Within 30 min.

2. Yes, but we don’t have to. In kubernetes, we can do rolling update while the service is running. Basically, we start a new pod and shutdown the old one.

3. Paritial failure is acceptable, if one pod is down, we can still serve the service. If all pods are down, all services are not available(It rarely happens).

4. We need about 10 min to restart all the services.

---
How should partial or total service failures be monitored and handled?

We use the built-in function in kubernetes to automatically restart a failed service. We offer http probe to help system monitor service states. There're a dashboard system to monitor the service states.

---
Do you need a recovery plan, or will it be covered by existing processes?

The recovery process is already implemented in kubernetes system.

---
Tracking/Documenting:
- How should the architecture support the process of problem reporting, tracking and fixing?

- What statistics do you need to keep about the site, and how will they be analysed?

- What instrumentation should be included in the design to measure performance, response times and availability?

- Should the architecture include a repository for statistical data?


1. Kubernetes redirects the stdout and stderr, we just print the logs to stdout and stderr, then we can find the log by `kubectl logs`

2. There're a dashboard system to monitor the service states.

3. Just use built-in time functions to measure function performance.

4. Yes, we can do that.

---
## Client-side Management

---
Who is the customer? (Internet or Intranet) – affects browser choice

The application will be deployed and accessible by Internet users using common and modern web browsers.

---
What is the level of the user’s skill?

Intermediate level to operate smartphones

---
What languages should the site support?

English

---
What are the user’s usage patterns? (search or browse)

Search-based route planning

---
How will the application maintain state?

Client-side maintains the logged-in user’s identity and credentials. Other user states are stored on server-side.

---
Is there a need to distribute application code, and if so, how will it be done?
Compiled production JavaScript will be distributed using Content Delivery Network services (e.g., Cloudflare).
How will the choice of client affect end-to-end response? (HTML, JavaScript, AJAX, JQuery, VBScript?)
Communications between the client and the server is done using Fetch API and WebSockets.
What are the different user interfaces needed?
Administration interface is required for User Management.


Network Management

Network Management Requirement
Impact on Project and how the Technical Architecture will address the impact
Will the solution involve the internet?
What protocols will be used? (HTTP? HTTPS? FTP? RMI? Messaging? Etc)


Yes, we use HTTPS

---
What about data, object and application placement?

projected transaction volumes, amount of data, interaction?

We use MySQL as our database. For each user, it takes about 1MiB of storage. And for each transaction, it takes about 10KiB of storage. Each user is expected to have 1-10 transactions per day in average.

---
What security functions are required/provided by chosen protocol?
Ievel of encryption will affect this, and also performance!


HTTPS provides the protection of traffic hijacking and the-man-in-middle attack. 
How does the network affect end-to-end response time?
About 20 – 50 ms


Server-side Management

Server-side Management Requirement
Impact on Project and how the Technical Architecture will address the impact
Single server or multiple servers? Peer-to-Peer? Sensors?
It is a centralized server.

Geographic location for servers?
It is hosted by vercel.app, they have multiple datacenters including EU.

End-user client to server, or server to server required also?
Client to server only
What security functions are required on the server?
mTLS and RSA with SHA256
How can impact of server on end-to-end response time be estimated, and catered for in the architecture?
We can use some HTTP performance test tools.



Application Logic

Server-side Management Requirement
Impact on Project and how the Technical Architecture will address the impact
Will site use client-side executables? What are their connectivity requirements?
Compiled JavaScript executables is executed and/or deployed on user devices.
Users should be connected during the period of usage.
How will application be split between client-side and server-side logic? (affects communications for validation etc/performance?)
The application is split by the API between client-side and server-side. User state storage, transitions, and validations are entirely performed on server-side.
Additional access security required?
N/A



Connectors

Server-side Management Requirement
Impact on Project and how the Technical Architecture will address the impact
What external systems, applications and (sensor) data does your project need to access?
We don’t access external systems.
How should data be transferred between different systems?
JSON data over HTTPS
How current does the information have to be? Use caches?
It’s a journey sharing application, for locations, we must use real-time data. For map data, we can use cache.
Is synchronous or asynchronous access required? Off-line OK?
We don’t have to strictly synchronize data on different users. It cannot run offline.
Is access to different operating systems, network protocols, application environments required? which connector? CICS? MQSeries? RPC?
Our service runs on one server, we still use HTTPS to connect each service – this makes the system modulized.
Are additional security policies required?
We use mTLS between different services.
Can scalability and performance requirements be predicted, and how will the project address these?
Yes. Because we use micro-service architecture by k8s, so the system is highly scalable.


Yes. Because we use micro service architecture(i.e. Kubernetes), so the system is highly scalable.