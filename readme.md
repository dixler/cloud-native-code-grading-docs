
# Table of Contents

1.  [Backend](#org9721824)
    1.  [Technologies](#orgdfb62d3)
        1.  [OAuth 2.0](#org9cadfae)
        2.  [Amazon Web Services](#org234d08f)
    2.  [Storage](#org0596735)
        1.  [DynamoDB](#org931f3e6)
        2.  [S3](#orgc3b905f)
        3.  [SQS](#org6c1dbad)
    3.  [APIs](#org42e8faf)
        1.  [Auth](#org7fd42e8)
        2.  [Upload](#orgc5d7c04)
        3.  [Modify Scoreboard](#orgaacd4e2)
        4.  [Modify Assignment](#orgdbaa80b)
        5.  [Query Database](#org14140ce)
        6.  [Grading Workflow](#org6cc4e90)
    4.  [Library Utilities](#org5d68535)
        1.  [[LOCAL] getNetId(token) -> string](#org43dcdda)
        2.  [[LOCAL] getTokenId(netid) -> string](#orga8c1f2f)
        3.  [[LOCAL] isOwner(token, assetId, assignmentId)](#org5ddde77)
        4.  [[LOCAL] isAdmin(token)](#orgec93cbd)
        5.  [[LOCAL][PRIVATE] insertDynamoDB(tablename, object) -> bool](#org159932d)
        6.  [[LOCAL][PRIVATE] retrieveDynamoDB(tablename, object) -> bool](#org73a6c4b)
    5.  [Callable APIs](#org94e9768)
        1.  [`Auth`](#org67934aa)
        2.  [`Create`](#org7ef991d)
        3.  [`Modify Assignment`](#org3b0bd19)
        4.  [`Query`](#org4f4fa23)
    6.  [Todolist](#org0b97285)
        1.  [<code>[1/1]</code> APIs](#org60bcaae)
        2.  [<code>[1/1]</code> API Gateway](#org57ca980)
2.  [Frontend](#orga70e723)
    1.  [Frames](#org4688e99)
        1.  [Scoreboard](#orgbe3428d)
        2.  [My Submissions](#org006fed4)
        3.  [Admin Interface](#org4025197)
        4.  [User Information](#org180f4a3)
    2.  [Storage](#orgf922eb8)
        1.  [S3](#orgabc805c)
    3.  [Actions](#org3878ac3)
        1.  [Submit Assignment](#org4ac9231)
        2.  [Create Assignment](#org0322688)
        3.  [Select Assignment](#org2044875)
        4.  [Log out](#org7bbb9e3)
    4.  [Provisioning](#orgfe3e1b7)
        1.  [S3 Bucket](#org0583243)
        2.  [AutoScaling group + Elastic Load Balancer](#orge003843)
        3.  [Instance Template](#orgb57ef63)
3.  [Grading Workflow](#orgb300b20)
    1.  [Security](#org912d538)
        1.  [Permissions:](#org958a0ff)
        2.  [User Classes:](#org0ba088d)
        3.  [Mitigations:](#org1360f3e)
    2.  [Constraints](#orgc73a08c)
    3.  [Grading AMI](#org394824a)
        1.  [gcc](#org4c46529)
        2.  [python](#orgf1c110f)
        3.  [make](#orgcca087f)
    4.  [Filesystem Layout](#org9c50ccb)
        1.  [Environment](#org6fa7583)
        2.  [DeploymentPackage.zip format](#org8b701e6)
    5.  [[PRIVATE][ABSTRACT] gradingWorkflow(netid, assignmentId, assetId)](#orgf60f7d7)
        1.  [Custom grading AMI](#org2f3699f)
        2.  [FETCH step](#org482da86)
        3.  [EXECUTE step](#org49dac89)
        4.  [WRITEBACK step](#org8db82a6)
    6.  [Todolist](#org5036f57)
        1.  [Provisioning](#orgdbd508b)
        2.  [Grading AMI](#org5cd78f1)
        3.  [[EXAMPLE]DeploymentPackage.zip](#orgd2bc3c9)
        4.  [retrieveAssets.py](#org50089c4)
        5.  [quarantine.py](#org16c495b)
        6.  [cloudgrade.sh](#org7e5208e)
        7.  [uploadScores.py](#org5e475e4)
        8.  [Monitoring CloudWatch Events](#org608a10d)
4.  [<code>[1/1]</code> Knowledge Gaps](#org7265cbe)
5.  [Reach Goals](#org6c0f220)
    1.  [Hidden Assignments for DEV/Prototyping](#org8d199db)
    2.  [Deleting Assignments](#org53f1680)
    3.  [Using user data to populate scripts into Grading Workflow Instance Template](#org814303f)
        1.  [Makes it more modular and faster to update scripts](#orgbf9afcf)
    4.  [<code>[0/3]</code> Reach Goals](#orgaa76e29)
        1.  [[PROTECTED] changeScoring(token, assignmentId, scoreBreakdown)](#orgf3e1741)
        2.  [[PROTECTED] changeDeploymentPackage(token, assignmentId, deploymentPackage)](#org452509e)
        3.  [[UNNECESSARY][PROTECTED] continueSession(token) -> bool](#org61dc418)



<a id="org9721824"></a>

# Backend


<a id="orgdfb62d3"></a>

## Technologies


<a id="org9cadfae"></a>

### OAuth 2.0

1.  Playground

    targets:
    <https://www.googleapis.com/auth/userinfo.email>
    <https://www.googleapis.com/auth/userinfo.profile>
    
    <https://accounts.google.com/o/oauth2/v2/auth?redirect_uri=https://developers.google.com/oauthplayground&prompt=consent&response_type=code&client_id=407408718192.apps.googleusercontent.com&scope=https://www.googleapis.com/auth/userinfo.email+https://www.googleapis.com/auth/userinfo.profile&access_type=offline>
    send the returned token to the application to request a token
    request:
    
        // Request
        POST /oauth2/v4/token HTTP/1.1
        Host: www.googleapis.com
        Content-length: 277
        content-type: application/x-www-form-urlencoded
        user-agent: google-oauth-playground
        code=4%2FAAC8yZdfe98kiiX6R-lojj5Dqe4HHtGkDPMlaFobNBn1ai4aHpmRhpyZvD49xxsyHJaD5of_1zf50I7KyMZeXyk&redirect_uri=https%3A%2F%2Fdevelopers.google.com%2Foauthplayground&client_id=407408718192.apps.googleusercontent.com&client_secret=************&scope=&grant_type=authorization_code
        
        // Response
        HTTP/1.1 200 OK
        Content-length: 1389
        X-xss-protection: 1; mode=block
        X-content-type-options: nosniff
        Transfer-encoding: chunked
        Expires: Mon, 01 Jan 1990 00:00:00 GMT
        Vary: Origin, X-Origin
        Server: GSE
        -content-encoding: gzip
        Pragma: no-cache
        Cache-control: no-cache, no-store, max-age=0, must-revalidate
        Date: Sun, 22 Jul 2018 16:26:54 GMT
        X-frame-options: SAMEORIGIN
        Alt-svc: quic=":443"; ma=2592000; v="44,43,39,35"
        Content-type: application/json; charset=UTF-8
        {
          // This is unique to the application
          "access_token": "ya29.GlsABqo0h39HIGVZS3nJ_ZxU73fIfK929JGLXve2CCFJXv89HNyGY-tJZAONtfJCo1nktLYMXENSfvmZKhPehprQki1ZmluG6iHNTWwSEyHTg8NIfSmDavrH9hQd", 
          "token_type": "Bearer", 
          "expires_in": 3600, 
          "refresh_token": "1/w2Ix95nfsB8x7dc_og4PhtvqxKKs10t2LW4-xvoZmxU", 
          // This is unique to the user
          "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6ImM2YWY3Y2FhMDg5NWZkMDFlNzc4ZGNlYWE3YTc5ODgzNDdkOGYyNWMifQ.eyJhenAiOiI0MDc0MDg3MTgxOTIuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiI0MDc0MDg3MTgxOTIuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMDA3NDA4MTcxMjQ4NjUwNDA2MTgiLCJoZCI6InVpYy5lZHUiLCJlbWFpbCI6ImtkaXhsZTJAdWljLmVkdSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJhdF9oYXNoIjoiengxT2dGeURTVV9JMlRLRl9WenRBUSIsImV4cCI6MTUzMjI4MDQxNCwiaXNzIjoiaHR0cHM6Ly9hY2NvdW50cy5nb29nbGUuY29tIiwiaWF0IjoxNTMyMjc2ODE0LCJuYW1lIjoiS3lsZSBEaXhsZXIiLCJwaWN0dXJlIjoiaHR0cHM6Ly9saDUuZ29vZ2xldXNlcmNvbnRlbnQuY29tLy1CZnZfY3NOUWRHWS9BQUFBQUFBQUFBSS9BQUFBQUFBQUFBQS9BQW5uWTdyUWZfVmE5NDFpZnZYMFMxdFViYl9qU2x6OVBRL3M5Ni1jL3Bob3RvLmpwZyIsImdpdmVuX25hbWUiOiJLeWxlIiwiZmFtaWx5X25hbWUiOiJEaXhsZXIiLCJsb2NhbGUiOiJlbiJ9.H9LJ8do7VqUepbYI4t2Mr1yrvTQJYvUnoy7XGdbXTdKUhHYG7VKpoWd76MkT77oujXQhpMJ60c_BYhCo8EpxufM75gYhVWhak4L1MOWIeHKwJFfRxu9kWVsgBtsAwsUKe4J4ZswA06bLQP10i9GB1Fjo0bimRaTfFyWTDdO_UfpMVw1QvsdPFUqhTGAmqHY3dBU3sNBMZcI3LtFCUVAlCR3-YFDsOvklyhRrwNV_oqDh6HoCL4IJGViI6LMpAdSJjNElFzitlEXUV8ET2Fw1FsAxs-PPKNmYVgI1119yYKmmm_KQJoZI5vwIZtzZKdzh6pnV7SRlWtcKASh-ynr0lg"
        }
    
    Now we can use the token to request user data. The token may time out, but we can use it on our end.
    We use the credential to retrieve user data:
    
    -   email(uic email)
    
        // Request
        GET /oauth2/v2/userinfo HTTP/1.1
        Host: www.googleapis.com
        Content-length: 0
        Authorization: Bearer ya29.glsabqo0h39higvzs3nj_zxu73fifk929jglxve2ccfrxv89hnyly-tjzaontfjco1nktlymxensfvmzkhpehprqkb1zmlug6ihntwwseyhtg8nifsmdavrh9hqd
        
        // Response
        HTTP/1.1 200 OK
        Content-length: 331
        X-xss-protection: 1; mode=block
        Content-location: https://www.googleapis.com/oauth2/v2/userinfo
        X-content-type-options: nosniff
        Transfer-encoding: chunked
        Expires: Mon, 01 Jan 1990 00:00:00 GMT
        Vary: Origin, X-Origin
        Server: GSE
        -content-encoding: gzip
        Pragma: no-cache
        Cache-control: no-cache, no-store, max-age=0, must-revalidate
        Date: Sun, 22 Jul 2018 16:29:04 GMT
        X-frame-options: SAMEORIGIN
        Alt-svc: quic=":443"; ma=2592000; v="44,43,39,35"
        Content-type: application/json; charset=UTF-8
        {
          "family_name": "Dixler", 
          "name": "Kyle Dixler", 
          "picture": "https://lh5.googleusercontent.com/-Bfv_csNQdGY/AAAAAAAAAAI/AAAAAAAAAAA/AAnnY7rQf_Va941ifvX0S1tUbb_jSlz9PQ/mo/photo.jpg", 
          "locale": "en", 
          "email": "kdixle2@uic.edu", 
          "given_name": "Kyle", 
          "id": "000000000000000000000", 
          "hd": "uic.edu", 
          "verified_email": true
        }
    
    Then we can store the cookie into our tokens-table and return it for our user to put in localStorage and to use as their token


<a id="org234d08f"></a>

### Amazon Web Services

1.  DynamoDB

2.  S3

3.  Lambda

4.  API Gateway

5.  EC2


<a id="org0596735"></a>

## Storage


<a id="org931f3e6"></a>

### DynamoDB

1.  tokens-table

    Stores the session credentials of our users
    put a lifecycle policy for these tokens
    
          //"token": "eyJhbGciOiJSUzI1NiIsImtpZCI6ImM2YWY3Y2FhMDg5NWZkMDFlNzc4ZGNlYWE3YTc5ODgzNDdkOGYyNWMifQ.eyJhenAiOiI0MDc0MDg3MTgxOTIuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiI0MDc0MDg3MTgxOTIuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMDA3NDA4MTcxMjQ4NjUwNDA2MTgiLCJoZCI6InVpYy5lZHUiLCJlbWFpbCI6ImtkaXhsZTJAdWljLmVkdSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJhdF9oYXNoIjoiengxT2dGeURTVV9JMlRLRl9WenRBUSIsImV4cCI6MTUzMjI4MDQxNCwiaXNzIjoiaHR0cHM6Ly9hY2NvdW50cy5nb29nbGUuY29tIiwiaWF0IjoxNTMyMjc2ODE0LCJuYW1lIjoiS3lsZSBEaXhsZXIiLCJwaWN0dXJlIjoiaHR0cHM6Ly9saDUuZ29vZ2xldXNlcmNvbnRlbnQuY29tLy1CZnZfY3NOUWRHWS9BQUFBQUFBQUFBSS9BQUFBQUFBQUFBQS9BQW5uWTdyUWZfVmE5NDFpZnZYMFMxdFViYl9qU2x6OVBRL3M5Ni1jL3Bob3RvLmpwZyIsImdpdmVuX25hbWUiOiJLeWxlIiwiZmFtaWx5X25hbWUiOiJEaXhsZXIiLCJsb2NhbGUiOiJlbiJ9.H9LJ8do7VqUepbYI4t2Mr1yrvTQJYvUnoy7XGdbXTdKUhHYG7VKpoWd76MkT77oujXQhpMJ60c_BYhCo8EpxufM75gYhVWhak4L1MOWIeHKwJFfRxu9kWVsgBtsAwsUKe4J4ZswA06bLQP10i9GB1Fjo0bimRaTfFyWTDdO_UfpMVw1QvsdPFUqhTGAmqHY3dBU3sNBMZcI3LtFCUVAlCR3-YFDsOvklyhRrwNV_oqDh6HoCL4IJGViI6LMpAdSJjNElFzitlEXUV8ET2Fw1FsAxs-PPKNmYVgI1119yYKmmm_KQJoZI5vwIZtzZKdzh6pnV7SRlWtcKASh-ynr0lg",
        {
          "token": "251",
          "netid": "kdixle2"
        }

2.  users-table

    Stores user credentials to allow for secrets, token is the tokenized user name generated 
    from the first time the user entered salted with their netid, we don't need to query the 
    board to reverse the tokenization.
    
        {
          "netid": "kdixle2",
          "admin": true,
          "uin": "666666666",
          "tokenid": "c1f8b18730737be44ea82b2a9ffd887729bd9ded1960e4b38bf49492fb0d1fa3"
        }

3.  submissions-table

    Store all of the submissions across every assignment
    
         // the asset id is generated using the submission time and the shasum of the submission
        {
          "assetId": "c1f8b18730737be44ea82b2a9ffd887729bd9ded1960e4b38bf49492fb0d1fa3",
          "netid": "kdixle2",
          "assignmentId": 0,
          "submissionDate": "2018-07-02 11:59",
          "state": "SUBMITTED"|"SCORED",
          "score": 100,
          "scoreBreakdown": {
            "some_field": 100,
            "other_field": 100,
            "my_field": 100,
            "field": 100,
            "feld": 100
          }
        }

4.  scoreboard-table

    we will be using the tokenid as the HASH key and the assignmentId as the RANGE key
    Store the highest scores for each student
    tokenid should be unique to each assignmentId
    
        {
          // the asset id is generated using the submission time and the shasum of the submission
          "assetId": "c1f8b18730737be44ea82b2a9ffd887729bd9ded1960e4b38bf49492fb0d1fa3",
          "assignmentId": 0,
          "tokenid": "c1f8b18730737be44ea82b2a9ffd887729bd9ded1960e4b38bf49492fb0d1fa3",
          "submissionDate": "2018-07-02 11:59",
          "state": "SUBMITTED"|"SCORED",
          "score": 100,
          "scoreBreakdown": {
            "some_field": 100,
            "other_field": 100,
            "my_field": 100,
            "field": 100,
            "feld": 100
          }
        }

5.  assignments-table

    Store the data for each assignment
    using [-1] as the metadata index
    
        {
          "assignmentId": 0,
          "assignmentName": "Linked Lists",
          "deadline": "2018-07-02 11:59",
          "score": 100,
          "scoreBreakdown": {
            // these are user defined and are up to the user to keep coherent
            "some_field": 100,
            "other_field": 100,
            "my_field": 100,
            "field": 100,
            "feld": 100
          }
        }

6.  configuration-table

    Store resource names and mappings to make this solution more generic
    
         {
           "CRN": 44322,
           "metadata": {
             "instructor": "John Lillis",
             "course_number": "251",
             "course_name": "Data Structures"
           },
           "s3": {
             "documents-bucket": "$bucketname"
           },
           "dynamodb": {
             "users-table": "$tablename",
             "assignments-table": "$tablename",
             "submissions-table": "$tablename"
           }
           "sqs": {
             "grading-queue": "$queuename"
           }
        }


<a id="orgc3b905f"></a>

### S3

1.  documents-bucket

    Stores everyone's submissions and the testing deploy packages.
    Namespace appears like this
    
    s3://documents-bucket.name/assignment.assignmentId/&#x2026;
    
    1.  Deployment Packages
    
        Deploy Packages are stored at:
        
            {$configuration-table[$CRN]["s3"]["documents-bucket"]}/{$assignments-table[$assignmentId]}/"deploymentPackage.zip"
            // uic-documents-bucket-44322-fa2018/0/deploymentPackage.zip
    
    2.  Assets
    
        Assets are stored at:
        
            {$configuration-table[$CRN]["s3"]["documents-bucket"]}/{$assignments-table[$assignmentId].assignmentId}/{sha256hash(asset+$submissionTime+$random)}
            // uic-documents-bucket-44322-fa2018/0/c1f8b18730737be44ea82b2a9ffd887729bd9ded1960e4b38bf49492fb0d1fa3
            // the lookup in the assignments table is done as a check to see if the assignment is valid otherwise an exception will be thrown


<a id="org6c1dbad"></a>

### SQS

1.  [FIFO] grading-queue

    stores the list of assets to be graded.
    using content based deduplication:
    
    -   Our assetIds should be unique thus our messages will be unique so we 
        should use extra protections
    
    1.  data format:
    
            {
              "assetId": "{assetId}",
              "assetBucket": "{documents-bucket-name}",
              "assignmentId": "{assignmentId}"
            }


<a id="org42e8faf"></a>

## APIs


<a id="org7fd42e8"></a>

### Auth

1.  [PUBLIC] login(authorizationCode) -> bearerToken

    Responsibility to get a new login token and associate it with the current user.
    If that user doesn't exist, make them exist and populate their users-table entry using the hash of the current time
    
    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   users-table
            
            2.  Write:
            
                -   tokens-table
                -   users-table

2.  [PROTECTED] continueSession(token) -> bool

    Responsibility to verify that the token is legitimate
    
    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table
            
            2.  Write:

3.  [PROTECTED] logout(token) -> null

    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
            2.  Write:
            
                -   tokens-table


<a id="orgc5d7c04"></a>

### Upload

1.  [PROTECTED] submitAssignment(token, assignmentId, asset)

    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table (validation)
                -   assignments-table (validation)
                -   users-table (retrieve netid)
            
            2.  Write:
            
                -   submissions-table
                
                provisions the submission-table entry to be modified later
                
                      // the asset id is generated using the submission time and the shasum of the submission
                    {
                      "assetId": "c1f8b18730737be44ea82b2a9ffd887729bd9ded1960e4b38bf49492fb0d1fa3",
                      "assignmentId": 0,
                      "netid": "kdixle2",
                      "state": "SUBMITTED"
                      "score": 0,
                      "scoreBreakdown": {}
                    }
                
                INDIRECTLY -> scoreboard-table
                  Workflow may modify this table if this is the new highest score
                
                    if score > currentScore:
                      updateScoreboard(netid, assetId, assignmentId)
        
        2.  S3:
        
            1.  Read:
            
            2.  Write:
            
                -   documents-bucket/$assignmentId/{sha256sum(asset+submissionTime+random)}
        
        3.  SQS:
        
            1.  Read:
            
            2.  Write:
            
                -   grading-queue

2.  [PROTECTED][ADMIN] createAssignment(token, assignmentName, deploymentPackage, deadline, scoreBreakdown)

    Need to make sure to validate that the user is an admin
    
    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table(validation isUser)
                -   users-table(validation isAdmin)
            
            2.  Write:
            
                -   assignments-table
                
                    {
                      "assignmentId": {GENERATED},
                      "assignmentName": $assignmentName,
                      "deadline": $deadline,
                      "score": $scoreBreakdown
                    }
                    // call params, base64 encoded
                    {
                     "token": "251",
                      "assignmentName": "Linked Lists",
                      "deadline": "2018-12-02 11:59",
                      "deploymentPackage": "I2luY2x1ZGUgPHN0ZGlvLmg+CiNpbmNsdWRlIDxzdGRsaWIuaD4KCmludCBtYWluKCl7CiAgIHByaW50ZigiSGVsbG8gV29ybGQhXG4iKTsKICAgcmV0dXJuIDA7Cn0K",
                      "scoreBreakdown": {
                       "score": 100,
                       "scoreBreakdown": {
                         "some_field": 100,
                         "other_field": 100,
                         "my_field": 100,
                         "field": 100,
                         "feld": 100
                         }
                       }
                    }
        
        2.  S3:
        
            1.  Read:
            
            2.  Write:
            
                -   documents-bucket/{assignments-table[assignmentId]}/deploymentPackage.zip

3.  [PRIVATE] enterScore(assetId, assignmentId, score, scoreBreakdown)

    Writes a score in for the submission.
    
    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
            2.  Write:
            
                -   submissions-table


<a id="orgaacd4e2"></a>

### Modify Scoreboard

1.  [PRIVATE] updateScoreboard(netid, assetId, assignmentId)

    1.  Trigger: submissions-table dynamodb stream[MODIFY]
    
        When the submissions-table is modified,
         if the $SUBMISSION.state is 'SCORED', then
           if the $SUBMISSION.score is larger than the current scoreboard submission for getTokenId($SUBMISSION.netid)
             replace current scoreboard entry with tokenizedSubmission($ENTRY)
        
        its the highest of 
    
    2.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   submissions-table
            
            2.  Write:
            
                -   scoreboard-table


<a id="orgdbaa80b"></a>

### Modify Assignment

1.  [PROTECTED][ADMIN] changeDeadline(token, assignmentId, deadline)

    What if it's being extended after it's due?
    
    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table
                -   users-table
            
            2.  Write:
            
                -   assignments-table
                
                    {
                      "assignmentId": assignmentId,
                      "deadline": $deadline
                    }

2.  [FUTURE] `[PROTECTED][ADMIN] changeScoring(token, assignmentId, scoreBreakdown)`

    Not my problem if the rubric is broken
    This may require a change of the deployment package
    
    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table
                -   users-table
            
            2.  Write:
            
                -   assignments-table
                
                    {
                      "assignmentId": assignmentId,
                      "deadline": $deadline
                    }

3.  [FUTURE] `[PROTECTED][ADMIN] changeDeploymentPackage(token, assignmentId, deploymentPackage)`

    Not my problem if the deployment package is broken. This is a future reach goal
    This may require a change of the scoring
    
    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table(validate isUser)
                -   users-table(validate isAdmin)
            
            2.  Write:
        
        2.  S3:
        
            1.  Read:
            
            2.  Write:
            
                -   documents-bucket/{assignments-table[assignmentId]}/deploymentPackage.zip


<a id="org14140ce"></a>

### Query Database

1.  [PROTECTED] getSubmissions(token, assignmentId) -> [submission, &#x2026;]

    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table
                -   assignments-table(validate)
                -   submissions-table
            
            2.  Write:

2.  [PROTECTED] getScoreboard(token, assignmentId) -> [tokenizedSubmission, &#x2026;]

    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table
                -   assignments-table
                -   scoreboard-table
            
            2.  Write:

3.  [PROTECTED] getAssignments(token) -> [assignment, &#x2026;]

    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table
                -   assignments-table
            
            2.  Write:

4.  [PROTECTED] getUser(token) -> user

    get user's credentials
    this will be used to determine if the session token is legitimate
    returns {} if invalid
    
    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   tokens-table
                -   users-table
            
            2.  Write:


<a id="org6cc4e90"></a>

### Grading Workflow

1.  [PRIVATE][SECRET] quarantine(bool, instance-id)

    Set quarantine on the calling asset&#x2026;
    This endpoint/function is essentially the sudo button that we have to hide
    
    1.  Assets
    
        1.  EC2:
        
            1.  Read:
            
            2.  Write:
            
                -   attach-role to instance
                -   detach-role from instance


<a id="org5d68535"></a>

## Library Utilities

We use environment variables since they are consistent given that we use terraform to provision


<a id="org43dcdda"></a>

### [LOCAL] getNetId(token) -> string

1.  Assets

    1.  DynamoDB:
    
        1.  Read:
        
            -   tokens-table
        
        2.  Write:


<a id="orga8c1f2f"></a>

### [LOCAL] getTokenId(netid) -> string

getTokenId by retrieving the tokenizedId of this user

1.  Constraints:

    Tokenizing by hashing the netid and uin fails since the netid is public information 
    and the uin is within 100,000,000, so it takes 100,000,000 local guesses to brute 
    force 1 user's token.
    Due to this, I've decided to assign tokens by hashing the time that they first logged in 
    and some salt if necessary
    
    1.  Assets
    
        1.  DynamoDB:
        
            1.  Read:
            
                -   users-table
            
            2.  Write:


<a id="org5ddde77"></a>

### [LOCAL] isOwner(token, assetId, assignmentId)

1.  Assets

    1.  DynamoDB:
    
        1.  Read:
        
            -   tokens-table
            -   users-table
            -   submissions-table
        
        2.  Write:


<a id="orgec93cbd"></a>

### [LOCAL] isAdmin(token)

1.  Assets

    1.  DynamoDB:
    
        1.  Read:
        
            -   tokens-table
            -   users-table
        
        2.  Write:


<a id="org159932d"></a>

### [LOCAL][PRIVATE] insertDynamoDB(tablename, object) -> bool

This is going to be extremely common

    def insertDynamoDB(tablename, json):
      client = boto3.client('dynamodb')
      table_name = os.environ[endpoint]
      # TODO


<a id="org73a6c4b"></a>

### [LOCAL][PRIVATE] retrieveDynamoDB(tablename, object) -> bool

returns {} if successful else None

    def retrieveDynamoDB(tablename, json):
      client = boto3.client('dynamodb')
      table_name = os.environ[endpoint]
      # TODO


<a id="org94e9768"></a>

## Callable APIs


<a id="org67934aa"></a>

### `Auth`

1.  [PUBLIC] login(authorizationCode) -> bearerToken

2.  [PROTECTED] logout(token) -> null


<a id="org7ef991d"></a>

### `Create`

1.  [PROTECTED] submitAssignment(token, assignmentId, asset)

2.  [PROTECTED] createAssignment(token, assignmentName, deploymentPackage, deadline, scoreBreakdown)


<a id="org3b0bd19"></a>

### `Modify Assignment`

1.  [PROTECTED] changeDeadline(token, assignmentId, deadline)

2.  [PROTECTED] changeScoring(token, assignmentId, scoreBreakdown)

3.  [PROTECTED] changeDeploymentPackage(token, assignmentId, deploymentPackage)


<a id="org4f4fa23"></a>

### `Query`

1.  [PROTECTED] getSubmissions(token, assignmentId) -> [submission, &#x2026;]

2.  [PROTECTED] getScoreboard(token, assignmentId) -> [tokenizedSubmission, &#x2026;]

3.  [PROTECTED] getAssignments(token) -> [assignment, &#x2026;]

4.  [PROTECTED] getUser(token) -> user


<a id="org0b97285"></a>

## Todolist


<a id="org60bcaae"></a>

### <code>[1/1]</code> APIs

1.  DONE [PUBLIC] login(authorizationCode) -> bearerToken


<a id="org57ca980"></a>

### <code>[1/1]</code> API Gateway

1.  DONE [PUBLIC] login(authorizationCode) -> bearerToken


<a id="orga70e723"></a>

# Frontend


<a id="org4688e99"></a>

## Frames


<a id="orgbe3428d"></a>

### Scoreboard

1.  API calls

    1.  getScoreboard(token, assignmentId)


<a id="org006fed4"></a>

### My Submissions

1.  API calls

    1.  getSubmissions(token, assignmentId)


<a id="org4025197"></a>

### Admin Interface

1.  Build an interface to create assignments


<a id="org180f4a3"></a>

### User Information

1.  API calls

    1.  getUser(token)


<a id="orgf922eb8"></a>

## Storage


<a id="orgabc805c"></a>

### S3

1.  frontend-assets-bucket


<a id="org3878ac3"></a>

## Actions


<a id="org4ac9231"></a>

### Submit Assignment

1.  API calls

    1.  submitAssignment(token, assignmentId, asset)


<a id="org0322688"></a>

### Create Assignment

1.  API calls

    1.  submitAssignment(token, assignmentId, asset)


<a id="org2044875"></a>

### Select Assignment

Changes currently selected item
Refreshes currently selected menu


<a id="org7bbb9e3"></a>

### Log out

1.  API calls

    1.  logout(token)


<a id="orgfe3e1b7"></a>

## TODO Provisioning


<a id="org0583243"></a>

### S3 Bucket

frontend-assets-bucket


<a id="orge003843"></a>

### AutoScaling group + Elastic Load Balancer


<a id="orgb57ef63"></a>

### Instance Template

1.  userdata to pull code and start node webserver


<a id="orgb300b20"></a>

# Grading Workflow


<a id="org912d538"></a>

## Security

This is the largest security hazard that we have to deal with.


<a id="org958a0ff"></a>

### Permissions:

The grading resource needs access to the following(either directly or indirectly):

-   S3
-   DynamoDB
-   SQS


<a id="org0ba088d"></a>

### User Classes:

1.  Instructor

    1.  Read
    
        1.  Local Filesystem
    
    2.  Write
    
        1.  Local Filesystem

2.  Student

    No access


<a id="org1360f3e"></a>

### Mitigations:

1.  Monitoring

    Monitor abnormally large files. Possibly a CloudWatch event pushing to an SNS notification.

2.  OS Permissions

    -   All code is run as a non-privileged user

3.  Firewall

    -   Remove internet gateway to prevent data exfiltration

4.  IAM Roles

    -   Invoke lambdas to assign and remove necessary permissions from the box as it needs them.
        Reduce the AWS permissions that the box has access to as the non privileged user runs 
        his code and return them in privileged mode.

5.  [PRIVATE][SECRET] quarantine(bool, instance-id)

    Set quarantine on the calling asset&#x2026;
    This endpoint/function is essentially the sudo button that we have to hide


<a id="orgc73a08c"></a>

## Constraints

1 grading job at a time per instance, due to quarantine race conditions.


<a id="org394824a"></a>

## Grading AMI

In order to perform our security role in this manner, we must provide features


<a id="org4c46529"></a>

### gcc


<a id="orgf1c110f"></a>

### python


<a id="orgcca087f"></a>

### make


<a id="org9c50ccb"></a>

## Filesystem Layout


<a id="org6fa7583"></a>

### Environment

/deploy/quarantine.py
/deploy/cloudgrade.sh
*deploy/uploadScores.py
/deploy/package.zip -> /deploy/package*


<a id="org8b701e6"></a>

### DeploymentPackage.zip format

package/
package/deploy.sh -> 
  should generate:
    package/score.json


<a id="orgf60f7d7"></a>

## [PRIVATE][ABSTRACT] gradingWorkflow(netid, assignmentId, assetId)

This grading workflow is run on a kubernetes cluster with docker containers running nodejs fetching jobs
from an AWS SQS queue. TODO have to make it so if a machine keels over, the asset still gets graded.


<a id="org2f3699f"></a>

### Custom grading AMI

In order to avoid extensive instructor permission, we will be using custom grading amis

    build assets:
    - gcc
    - g++
    - make
    - gnu coreutils
    grading assets:
    - aws-cli
    - python
    - python-boto3
    users:
    - user #unprivileged user


<a id="org482da86"></a>

### FETCH step

    const endpoints = configuration-table['HARDCODED-CRN']; // TODO Generalize
    const submissionsTable = endpoints['submissions-table'];
    const documentsBucket = endpoints['documents-bucket'];
    const gradingQueue = endpoints['grading-queue'];
    
    // to get job information
    const job = gradingQueue.retrieveJob()
    gradingQueue.sendReceipt()
    /*
    // job:
     {
       "netid": "$netid",
       "assignmentId": "$assignmentId",
       "assetId": "$assetId"
     }
    */
    
    const submission = submissionsTable[assetId];
    const deployPackage = 'documents-bucket/' + submissions-table[assetId].assignmentId + '/deployPackage.zip'
    const asset = 'documents-bucket/' + submissions-table[assetId].assignmentId + '/' + submission.assetId;
    retrieveAssets(deployPackage); // downloads the testing package
    retrieveAssets(asset); // downloads the asset to be evaluated
    executeGradingScript('/deploy/cloudgrade.sh', job) // kicks off the grading script to generate ./package/score.json and then will call a script to upload the scores
    // job is not private data
    // SHELL CMDLINE arguments are visible to users


<a id="org49dac89"></a>

### EXECUTE step

    #!/bin/bash
    #filename: /deploy/cloudgrade.sh
    # STORE ENDPOINTS
    cd /deploy/
    # set ./deploy/package owner to user
    # will cause this package to get reaped after upload
    chgrp user ./package -R
    chown user ./package -R
    chmod 700 ./package -R
    
    cd ./package
    sudo -u user ./deploy.sh
    cd /deploy
    
    # "/deploy/package/score.json" contains the score that should match the scoreBreakdown object
    ./uploadScores.py "$1" "$(cat ./package/score.json)"
    
    # clean up all files by user to maintain the clean state
    find / -user 'user' -exec rm -fr {} \;
    
    exit


<a id="org8db82a6"></a>

### WRITEBACK step

1.  uploadScores.py(job, score)

    submissionsTableEndpoint is required and can be retrieved from the configuration-table
    insert the score and scoreBreakdown and 'SCORED' into the submissions-table[assetId]
    
        // package/score.json
        {
          "score": 100,
          "scoreBreakdown": {
            // these are user defined and are up to the user to keep coherent
            "some_field": 100,
            "other_field": 100,
            "my_field": 100,
            "field": 100,
            "feld": 100
          }
        }


<a id="org5036f57"></a>

## TODO Todolist


<a id="orgdbd508b"></a>

### DONE Provisioning

1.  Grading AMI

2.  Subnet

3.  Instance Template

    1.  Security Groups

4.  AutoScaling group + Elastic Load Balancer

5.  <code>[0/2]</code> IAM roles

    1.  TODO [EC2] Quarantine Invoke Role
    
    2.  TODO [EC2] UnQuarantined Role


<a id="org5cd78f1"></a>

### DONE Grading AMI


<a id="orgd2bc3c9"></a>

### DONE [EXAMPLE]DeploymentPackage.zip


<a id="org50089c4"></a>

### DONE retrieveAssets.py


<a id="org16c495b"></a>

### DONE quarantine.py


<a id="org7e5208e"></a>

### DONE cloudgrade.sh


<a id="org5e475e4"></a>

### DONE uploadScores.py


<a id="org608a10d"></a>

### TODO Monitoring CloudWatch Events


<a id="org7265cbe"></a>

# <code>[1/1]</code> Knowledge Gaps

-   [X] OAuth implementation


<a id="org6c0f220"></a>

# Reach Goals


<a id="org8d199db"></a>

## Hidden Assignments for DEV/Prototyping


<a id="org53f1680"></a>

## Deleting Assignments


<a id="org814303f"></a>

## Using user data to populate scripts into Grading Workflow Instance Template


<a id="orgbf9afcf"></a>

### Makes it more modular and faster to update scripts


<a id="orgaa76e29"></a>

## <code>[0/3]</code> Reach Goals


<a id="orgf3e1741"></a>

### TODO [PROTECTED] changeScoring(token, assignmentId, scoreBreakdown)


<a id="org452509e"></a>

### TODO [PROTECTED] changeDeploymentPackage(token, assignmentId, deploymentPackage)


<a id="org61dc418"></a>

### TODO [UNNECESSARY][PROTECTED] continueSession(token) -> bool

