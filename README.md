# API-Automation-Testing-Framework
Overview
This repository contains an API testing framework for automating the testing of Alyke application APIs, focusing on authentication flows, user management, and reporting functionality. The framework provides comprehensive test coverage with both positive and negative scenarios, response validation, schema verification, and performance metrics.
Tech Stack

Java: Programming language
REST Assured: Library for API testing
TestNG: Test management and reporting
Jackson/Gson: JSON handling and schema validation
Maven: Dependency management and build automation
Extent Reports: Detailed test reporting
Log4j: Logging infrastructure

Project Structure
api-automation-framework/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── alyke/
│   │   │           ├── config/           # Configuration files
│   │   │           ├── constants/        # API endpoints and constants
│   │   │           ├── models/           # POJO classes for request/response
│   │   │           ├── utils/            # Utility helpers
│   │   │           │   ├── AuthUtils.java
│   │   │           │   ├── RestUtils.java
│   │   │           │   ├── SchemaValidator.java
│   │   │           │   └── RequestBuilder.java
│   │   │           └── reporting/        # Custom reporting functionality
│   │   └── resources/
│   │       ├── config.properties         # Environment configuration
│   │       ├── schemas/                  # JSON schema files for validation
│   │       └── testdata/                 # Test data files
│   └── test/
│       ├── java/
│       │   └── com/
│       │       └── alyke/
│       │           ├── tests/
│       │           │   ├── auth/         # Authentication API tests
│       │           │   │   ├── RequestOtpTests.java
│       │           │   │   └── CheckOtpTests.java
│       │           │   └── users/        # User-related API tests
│       │           │       ├── UserDetailTests.java
│       │           │       └── ReportUserTests.java
│       │           └── suites/           # Test suites configuration
│       └── resources/
│           └── testng.xml                # TestNG configuration
├── test-output/                          # Test reports
├── pom.xml                               # Maven dependencies
└── README.md                             # Project documentation
Features

Comprehensive Test Coverage:

Positive and negative test scenarios for all APIs
Response structure validation using JSON schema
Response time validation (<1 second threshold)
Proper error handling and validation


Authentication Management:

Automatic handling of bearer tokens
Token extraction and reuse
Header management (Authentication, region, version-code)


Reporting:

Detailed HTML reports with Extent Reports
Response time tracking and performance metrics
Request/response logging for debugging


Easy Configuration:

Environment-specific configurations
Test data management
Reusable components



API Coverage
1. Request OTP API

Endpoint: https://api.joinalyke.com/api/auth/requestOtp
Method: POST
Tests:

Positive test with valid payload
Negative test with missing fields
Response time validation (<1 sec)
Response schema validation



2. Check OTP API

Endpoint: https://api.joinalyke.com/api/auth/checkOtp
Method: POST
Tests:

Positive test with valid OTP
Negative test with incorrect OTP
Token extraction and validation
Response time validation



3. Get User Detail API

Endpoint: https://api.joinalyke.com/api/users/user-detail
Method: GET
Tests:

Positive test with valid userId
Negative test with non-existent user
Authentication header validation



4. Report User API

Endpoint: https://api.joinalyke.com/api/users/reportUser
Method: PATCH
Tests:

Positive test with valid report data
Negative test with missing required fields
Response structure validation



Setup Instructions
Prerequisites

Java JDK 11 or higher
Maven 3.6.3 or higher
Git

Installation Steps

Clone the repository:
bashgit clone https://github.com/yourusername/api-automation-framework.git
cd api-automation-framework

Install dependencies:
bashmvn clean install -DskipTests

Configure environment:

Update src/main/resources/config.properties with appropriate values



Running Tests
bash# Run all tests
mvn clean test

# Run specific test class
mvn clean test -Dtest=RequestOtpTests

# Run with specific suite
mvn clean test -DsuiteXmlFile=src/test/resources/testng.xml
Sample Test Results
Test execution generates detailed reports in the test-output directory:
===============================================
Test Suite: API Automation Suite
Total tests run: 8, Passes: 7, Failures: 1, Skips: 0
===============================================

PASSED: testValidOtpRequest
Response Time: 345ms
Status Code: 200

PASSED: testValidOtpCheck
Response Time: 412ms
Status Code: 200
Token successfully extracted

PASSED: testGetValidUser
Response Time: 278ms
Status Code: 200

FAILED: testReportWithMissingField
Expected Status Code: 400 but found: 500
Response Time: 389ms
Error Message: Internal Server Error
Key Implementation Details
Authentication Handling
java// Example of token extraction from checkOTP response
public static void extractAndStoreToken(Response response) {
    try {
        String token = response.jsonPath().getString("data.token");
        if (token != null && !token.isEmpty()) {
            PropertyManager.setProperty("auth.token", token);
            logger.info("Authentication token successfully extracted and stored");
        } else {
            logger.error("Token not found in the response");
        }
    } catch (Exception e) {
        logger.error("Error extracting token: " + e.getMessage());
    }
}
Request Building with Headers
javapublic static RequestSpecification buildAuthenticatedRequest() {
    String token = PropertyManager.getProperty("auth.token");
    return RestAssured.given()
        .header("Authentication", "Bearer " + token)
        .header("region", "regionTwo")
        .header("version-code", "2.0.0")
        .contentType(ContentType.JSON)
        .accept(ContentType.JSON)
        .log().ifValidationFails();
}
Response Validation
javapublic static void validateResponse(Response response, int expectedStatusCode, long maxResponseTime, String schemaPath) {
    // Status code validation
    Assert.assertEquals(response.getStatusCode(), expectedStatusCode, 
        "Status code validation failed");
    
    // Response time validation
    Assert.assertTrue(response.getTime() < maxResponseTime, 
        "Response time exceeded " + maxResponseTime + "ms: " + response.getTime() + "ms");
    
    // Schema validation if provided
    if (schemaPath != null && !schemaPath.isEmpty()) {
        try {
            response.then().assertThat()
                .body(JsonSchemaValidator.matchesJsonSchemaInClasspath(schemaPath));
        } catch (Exception e) {
            Assert.fail("Schema validation failed: " + e.getMessage());
        }
    }
}
Continuous Integration
This framework is designed to integrate with CI/CD pipelines:

Jenkins configuration included
GitHub Actions workflow ready
Supports headless execution in containerized environments

Contributing

Fork the repository
Create feature branch (git checkout -b feature/amazing-feature)
Commit changes (git commit -m 'Add amazing feature')
Push to branch (git push origin feature/amazing-feature)
Open a Pull Request

License
This project is licensed under the MIT License - see the LICENSE file for details.
