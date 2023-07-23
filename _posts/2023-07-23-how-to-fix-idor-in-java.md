---
layout: post
title: "Detecting & Fixing Insecure Direct Object References (IDOR) in Java Applications"
author: rohitcoder
categories: [ IDOR ]
tags: [ java, idor ]
image: https://i.imgur.com/SFM9fNm.png
featured: false
hidden: false
---

**Introduction**
-----------------
Insecure Direct Object References (IDOR) is a critical security vulnerability that can lead to unauthorized access to sensitive resources in web applications. In this blog post, we will discuss real-life-like IDOR scenarios in Java applications, with some vulnerabilities being relatively easy to find and others being more challenging. We will provide code snippets representing the vulnerable scenarios and demonstrate how to fix them securely. Additionally, we will explore different approaches in the Secure Software Development Life Cycle (SSDLC) to mitigate IDOR vulnerabilities during early stages, such as PRD security reviews.


**Understanding IDOR**
-----------------------
Insecure Direct Object References occur when an application allows direct access to internal resources (e.g., files, database records) without proper access controls. Attackers can exploit IDOR to access sensitive information or perform actions that should be restricted to certain users, leading to data leaks, unauthorized disclosure, or even full system compromise.

**Impact of IDOR**
-------------------
Consider a simple Java web application that displays user details based on the provided user ID. The application allows users to view their profile by providing their ID as a URL parameter.

```java
public class UserProfileController extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String userIdParam = request.getParameter("id");
        int userId = Integer.parseInt(userIdParam);
        
        User currentUser = getCurrentUserFromDatabase(); // Assume this fetches user details from the database based on the session.
        User requestedUser = getUserFromDatabase(userId); // Vulnerability: No access control check.
        
        if (requestedUser != null && currentUser != null && requestedUser.getId() == currentUser.getId()) {
            // Display the user profile
            response.getWriter().println("Welcome, " + requestedUser.getName() + "!");
            // Display other user details
        } else {
            response.sendError(HttpServletResponse.SC_FORBIDDEN, "Access Denied");
        }
    }
}
```

In this example, the application fetches user details based on the provided user ID without checking if the current user has permission to access that information. An attacker could manipulate the "id" parameter in the URL to view other users' profiles.


**Fixing Case Study 1: Implement Proper Access Control**
---------------------------------------
To fix the IDOR vulnerability in Case Study 1, we need to ensure that users can only access their own profiles. We can achieve this by implementing proper access control checks.

```java
public class UserProfileController extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String userIdParam = request.getParameter("id");
        int userId = Integer.parseInt(userIdParam);
        
        User currentUser = getCurrentUserFromDatabase(); // Assume this fetches user details from the database based on the session.
        User requestedUser = getUserFromDatabase(userId);
        
        if (currentUser != null && requestedUser != null && currentUser.getId() == requestedUser.getId()) {
            // Display the user profile
            response.getWriter().println("Welcome, " + requestedUser.getName() + "!");
            // Display other user details
        } else {
            response.sendError(HttpServletResponse.SC_FORBIDDEN, "Access Denied");
        }
    }
}
```

In the fixed code, we verify that the current user (the one making the request) is the same as the user whose profile is being accessed. If not, the server responds with a "403 Forbidden" status code.


**Case Study 2: Complex IDOR Vulnerability**
-----------------------------------
Let's explore a more complex scenario where the application has different types of resources and access levels. Consider a file-sharing application written in Java, where users can upload files and share them with specific groups or individual users. Each file has a unique identifier (file ID) associated with it.

```java
public class FileSharingController extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String fileIdParam = request.getParameter("fileId");
        int fileId = Integer.parseInt(fileIdParam);
        
        File requestedFile = getFileFromDatabase(fileId); // Vulnerability: No access control check.
        
        if (requestedFile != null) {
            User currentUser = getCurrentUserFromDatabase(); // Assume this fetches user details from the database based on the session.
            boolean authorized = isUserAuthorized(requestedFile, currentUser);
            if (authorized) {
                // Serve the file for download
                // ...
            } else {
                response.sendError(HttpServletResponse.SC_FORBIDDEN, "Access Denied");
            }
        } else {
            response.sendError(HttpServletResponse.SC_NOT_FOUND, "File not found");
        }
    }
}
```

In this scenario, the application serves files based on the provided "fileId" URL parameter. However, there is no access control check to ensure that the current user has permission to access the file.

**Fixing Case Study 2: Implement Proper Authorization**
-----------------------------------
To fix the complex IDOR vulnerability in Case Study 2, we need to implement proper authorization checks to ensure that users can only access files they have permission to view.

```java
public class FileSharingController extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String fileIdParam = request.getParameter("fileId");
        int fileId = Integer.parseInt(fileIdParam);
        
        File requestedFile = getFileFromDatabase(fileId);
        
        if (requestedFile != null) {
            User currentUser = getCurrentUserFromDatabase(); // Assume this fetches user details from the database based on the session.
            boolean authorized = isUserAuthorized(requestedFile, currentUser);
            if (authorized) {
                // Serve the file for download
                // ...
            } else {
                response.sendError(HttpServletResponse.SC_FORBIDDEN, "Access Denied");
            }
        } else {
            response.sendError(HttpServletResponse.SC_NOT_FOUND, "File not found");
        }
    }

    private boolean isUserAuthorized(File file, User user) {
        // Implement the logic to check if the user is authorized to access the file.
        // Check if the user is the owner or if they belong to a group with access.
        // ...
    }
}
```

In the fixed code, we've added a separate method isUserAuthorized() to check if the current user is authorized to access the requested file. The method implements the necessary logic to validate permissions based on user roles, file ownership, and group access.

**SSDLC Approach to Mitigate IDOR Vulnerabilities**
---------------------------------------------------
In the Secure Software Development Life Cycle (SSDLC), various stages can help identify and mitigate IDOR vulnerabilities before they become critical issues. Let's explore some of these stages:

 - **PRD Security Review:** During the Product Requirements Document (PRD) review, security experts and stakeholders can assess the application's design and architecture to identify potential IDOR risks. Emphasis should be placed on access control mechanisms and data flow diagrams.

 - **Threat Modeling:** Conduct threat modeling sessions to identify potential IDOR threats early in the development process. By mapping out application components and data flows, developers can proactively implement security controls and access restrictions.

 - **Code Reviews and Static Analysis:** Perform code reviews and utilize static analysis tools to identify code patterns susceptible to IDOR vulnerabilities. Reviewers can check for direct object references without proper authorization checks.

 - **Security Testing:** Perform dynamic security testing, including penetration testing and vulnerability scanning, to simulate real-world attack scenarios and identify IDOR vulnerabilities in the application.

 - **User Permissions and Role-Based Access Control (RBAC):** Implement strong user permissions and RBAC mechanisms to ensure that each user can only access resources they are authorized to use.

 - **Secure API Design:** If the application provides APIs, ensure that they include access controls and user authentication mechanisms to prevent unauthorized access to sensitive data.

 - **Least Privilege Principle:** Apply the least privilege principle to limit access to resources and functions to only what is necessary for each user role.

**Conclusion**
---------------
Insecure Direct Object References (IDOR) are dangerous vulnerabilities that can lead to unauthorized access to sensitive resources. In this blog post, we explored real-life-like IDOR scenarios in Java applications and demonstrated how to fix them securely. By implementing proper access controls, authorization checks, and following the Secure Software Development Life Cycle (SSDLC) best practices, developers can effectively mitigate IDOR vulnerabilities and ensure the security of their web applications. Regular security testing and proactive security measures play a crucial role in building robust and secure Java applications.
