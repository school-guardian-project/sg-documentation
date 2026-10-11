# Corrective, Preventive, and Software Improvement Actions
## Guardian Escolar

**Development team:**  
Johan Smith Santamaria Fernández  
Sharik Dayanna Rojas Ibarra  
Juan Pablo Chala Ramírez  

**Institution:** National Learning Service (SENA)  
**Training group:** ADSO-3145556  
**Training center:** Neiva, Huila, Colombia  
**Instructor:** José de Jesús Motta Vargas  
**Date:** October 6, 2026  

---

## Version control

| Version | Date | Author | Description |
|---|---|---|---|
| 1.1 | October 11, 2026 | Guardian Escolar Team | Revised the document to record actions and decisions reported by the team and remove unconfirmed figures and scenarios. |

## 1. Introduction

During the development of Guardian Escolar, the team identified situations that led to changes in interface design, adjustments to development practices, stronger data validation, remediation of an accidental credential exposure, and decisions about the technologies used. The team also received proposals that were evaluated but did not necessarily become system changes.

This document records the corrective, preventive, and improvement actions reported by the team. Its purpose is to document findings, decisions, reported status, and items requiring further verification. The statuses below are based on the team's statements and do not replace technical evidence from tests, commits, or reviews.

## 2. Objectives

- Document the main issues, corrections, decisions, and improvements that occurred during Guardian Escolar development.
- Distinguish completed corrections from proposals the team decided not to adopt.
- Record the reported status of each action without claiming unconfirmed results.
- Identify outstanding checks that can support software security, quality, and maintainability.

## 3. Scope

This register covers web design, the main page, responsive layouts, Git practices, removal of mock data, credential security, form validation, super-administrator management, mobile application migration, GPS integration, and database engine selection.

This document does not, by itself, prove that all system features are complete or that every action has formal effectiveness-test evidence. Additional verification is explicitly identified where needed.

## 4. Classification criteria

- **Corrective action:** addresses a problem or nonconformity that has already occurred.
- **Preventive action:** seeks to reduce the likelihood of a potential problem occurring or recurring.
- **Improvement action:** introduces a change intended to improve quality, usability, maintainability, or development practices.
- **Evaluated proposal not adopted:** records an alternative considered by the team but not implemented. It is not classified as a completed correction.

An action may both correct a problem and help prevent its recurrence. The classification in the table describes the change's primary purpose.

## 5. Action register

| ID | Finding or situation | Type | Action or decision | Reported status |
|---|---|---|---|---|
| AC-01 | The initial design used yellow as its primary color. During the first review, the team was asked to change it to blue. | Improvement | The interfaces were redesigned and blue was adopted as the system's primary color. | Corrected |
| AC-02 | The main page had a simple presentation and included a section with an explanatory video about the program. | Improvement | The main page was redesigned for a more professional presentation, and the video section was removed. | Corrected |
| AC-03 | It was suggested that tracking should use a mobile phone instead of tracking the vehicle directly. | Evaluated proposal; design decision | The team decided to keep vehicle tracking because the goal is to identify the school bus's location and route, rather than only a person's location. | Proposal not adopted |
| AC-04 | Commit messages did not follow a consistent convention, and some described changes too generally. | Development-process improvement | Commit messages were adjusted to use English and describe changes more precisely, following a more consistent structure. | Corrected in GitHub |
| AC-05 | Mock data was still being used at a stage when the affected features needed to move toward data from real services. | Corrective | The identified mock data was removed, and work progressed toward using information from the relevant APIs and services. | Reported as corrected; verify real data consumption in the affected features |
| AC-06 | Responsive design improvements were needed for the administrator and super-administrator dashboards, as well as screens for password recovery or changes and phone-number and email updates. | Improvement | Design and responsive-layout adjustments were made to the affected interfaces. | Corrected |
| AC-07 | Environment credentials, including database credentials, were accidentally published in GitHub history. | Corrective and preventive | The exposed credentials were changed, and `.env` was added to `.gitignore` to prevent accidental inclusion in new commits. | Measures completed; verify that old secrets are no longer usable and assess the published history |
| AC-08 | Web and mobile forms allowed attempts to submit empty data or inappropriate characters and formats. | Corrective and preventive | Frontend validations and backend controls were added. | Reported as corrected |
| AC-09 | The initial approach to school management and administrator assignment presented operational difficulties. | Functional improvement | A super-administrator role and section were added to centralize the relevant school and administrator management functions. | Corrected |
| AC-10 | The initial mobile implementation with .NET MAUI was difficult for the team to manage. | Technology improvement | The mobile application was migrated to React Native with Expo. | Migration completed |
| AC-11 | The IoT/GPS device existed, but its integration with the project had not yet been fully implemented. | Corrective and improvement | Integration between the GPS device and the backend progressed. | Partially implemented; real-time route tracking still needs to be completed or verified |
| AC-12 | PostgreSQL was initially proposed, but the team decided to use a different database engine. | Technology improvement | Microsoft SQL Server was adopted as the project's database engine. | Change made |

## 6. Description of actions

### 6.1. Interface and main-page redesign

During the first project review, the team was asked to change the primary color from yellow to blue. This required redesigning the interfaces to maintain a consistent visual identity. During the same process, the main page was redesigned to look more professional, and the explanatory-video section was removed.

### 6.2. Standardizing commit messages

The team identified the need for commit messages to describe changes more clearly and follow a consistent convention. Their structure was adjusted to use English and more specific descriptions. The goal is to make the repository history easier to read and changes easier to understand.

### 6.3. Removal of mock data

Mock data was used during an earlier stage to build or test interfaces. Later, the team identified the need for affected features to move toward real information from the relevant APIs and services. The team reported that the identified mock data was removed. To confirm functional closure, the affected screens and operations should be checked to ensure they consume real data and do not retain demonstration values.

### 6.4. Credential security

The team detected that configuration credentials, including database credentials, had accidentally been published in GitHub history. The team reported changing the credentials and adding `.env` to `.gitignore`.

These measures reduce the risk of the exposed credentials being used and help prevent the file from being accidentally included in future commits. However, `.gitignore` does not by itself remove secrets from earlier Git commits. As a security follow-up, the team should verify that the old credentials are no longer valid and assess whether the secrets need to be removed from the published history.

### 6.5. Data validation in web, mobile, and backend components

The team identified that forms allowed attempts to submit empty fields or values with inappropriate formats. Frontend validations and backend controls were added. Verification should include empty inputs, invalid formats, and direct requests to the backend because client-side validation does not replace server-side validation.

### 6.6. Addition of the super-administrator role

The initial management of schools and administrator assignment presented difficulties. To address this, the team added a super-administrator role and section with management functions consistent with the defined scope of that role.

### 6.7. Mobile application migration

The mobile application was initially started with .NET MAUI. Because the team experienced difficulties working with that technology, it decided to migrate to React Native with Expo. The reported status is that the migration was completed.

### 6.8. GPS integration

At an early stage, the GPS/IoT device was not fully integrated into the project. The team reported that integration with the backend and physical device has since progressed, but real-time route tracking still needs to be completed or verified. Therefore, this action is recorded as partially implemented rather than complete.

### 6.9. Database engine change

The initial proposal considered PostgreSQL. The team later decided to use **Microsoft SQL Server**. Technical documentation and project configuration should use this name consistently. Microsoft SQL Server must not be confused with MySQL.

### 6.10. Proposal to track a mobile phone

A proposal was made to use a mobile phone for tracking instead of locating the tracking device in the vehicle. The team decided not to adopt this alternative because Guardian Escolar is intended to track the school bus route. This entry documents a design decision, not an implemented correction.

## 7. Follow-up and verification

The following items are recommended checks. They do not imply that the team failed to perform the actions described; they are intended to verify effectiveness and keep the document aligned with the project's actual status.

| Related ID | Recommended verification | Verification criterion |
|---|---|---|
| AC-05 | Review the features from which mock data was removed. | Displayed information and operations come from the relevant APIs or services rather than demonstration values. |
| AC-07 | Confirm credential rotation and review the published history. | Old credentials are no longer valid; determine whether secrets need to be removed from history. |
| AC-08 | Test web and mobile forms, as well as direct requests to the backend. | Empty or invalid data is appropriately rejected by both the client and server. |
| AC-11 | Test position reception and the vehicle route. | Real-time route behavior is verified before the integration is marked complete. |
| AC-12 | Compare the documentation with the project's configuration and scripts. | The documented technology matches Microsoft SQL Server in the relevant environment. |

When concrete evidence becomes available, such as commits, test results, or screenshots of the feature, it may be added to the register. Do not invent closure dates, responsible parties, test results, or commit references.

## 8. Conclusions

The recorded actions reflect changes made to Guardian Escolar's interface design, version-control practices, data usage, security, validation, role management, mobile technology, GPS, and database engine.

The register distinguishes changes reported as corrected, design decisions that did not result in a system modification, and features that still require completion or verification. In particular, GPS tracking remains partially implemented according to the available information, and the credential incident requires consideration of the fact that `.gitignore` does not remove secrets from earlier commits.

The document should be updated when new corrections or verification evidence becomes available, maintaining a clear distinction between completed actions, proposals not adopted, and outstanding tasks.

---

## Appendix A. Evidence that may be associated with actions

| Evidence type | Purpose |
|---|---|
| Commit or pull request | Show the change associated with design, validation, migration, or configuration. |
| Screenshot | Illustrate the state of an interface before or after an adjustment. |
| Test result | Confirm validation, API consumption, or GPS behavior. |
| Credential-rotation evidence | Confirm that exposed credentials were replaced without publishing the new secrets. |
| Technical configuration | Confirm the database technology in use. |

Do not include passwords, tokens, connection strings containing secrets, or other sensitive data in the evidence.
