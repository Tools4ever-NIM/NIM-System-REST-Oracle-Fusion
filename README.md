# Oracle Fusion Cloud HCM

Supports employee creation and updates, plus collection of the tables below.

Configure the Oracle Fusion hostname (for example, `servername.fa.us2.oraclecloud.com`, without `https://` or a path), username, and password. The connector uses Basic authentication over HTTPS. The account needs permission for each selected resource and employee write operation. Test Connection reads one Worker; permissions for other tables and actions must be checked in the target environment.

| Table | Resource | Key |
| --- | --- | --- |
| Employees | `/emps` | `PersonId` |
| Locations | `/locations` | `LocationId` |
| Departments | `/organizations` | `OrganizationId` |
| Jobs | `/jobs` | `JobId` |
| Positions | `/positions` | `PositionId` |
| Workers | `/workers` | `PersonId` |
| Assignments | `/emps/{empsUniqID}/child/assignments` | `AssignmentId` |

All paths use `/hcmRestApi/resources/11.13.18.05`. Tables expose documented scalar attributes; flexfields are not expanded. Identifiers are imported as strings. Effective-dated resources use Oracle's default current-date view.

Employees also collects an `Employees_links` child table containing Oracle's self links, with `Employees_PersonId`, `rel`, and `EmployeeUrl`. Keep these columns selected. Assignments iterates those self links and paginates each employee's assignments independently, retaining `PersonId` on each assignment. This uses Oracle's returned hash key without constructing one from `PersonId`. The REST base URL is intentionally empty, and collection calls use full URLs so NIM can follow these self links.

`EmployeesCreate` sends `POST /emps`. Its mandatory fields match Oracle's published request schema: `CitizenshipLegislationCode`, `CitizenshipStatus`, `Country`, `EffectiveStartDate`, `HireDate`, `HomeFaxNumber`, `HomePhoneNumber`, `LastName`, `LegalEntityId`, `NationalId`, `NationalIdCountry`, `NationalIdType`, `WorkEmail`, `WorkFaxNumber`, `WorkMobilePhoneNumber`, and `WorkPhoneNumber`. Other supported scalar request fields are optional. Tenant rules can impose additional requirements. Nested assignment creation is not included.

`EmployeesUpdate` sends `PATCH /emps/{empsUniqID}`. Map the required, input-only `empsUniqID` argument from the final path segment of `Employees_links.EmployeeUrl`, joining `Employees_links.Employees_PersonId` to `Employees.PersonId`. For example, a self URL ending in `/emps/00020000000EACED...` supplies `00020000000EACED...`. Preserve Oracle's returned hash key; do not substitute `PersonId` or generate a hash. NIM removes this path argument from the request body. Only scalar fields supported by Oracle's update schema are available to map; read-only fields are prohibited.

Employee Update Effective-Of defaults to `RangeMode=CORRECTION`, which corrects the existing effective record. To make a dated update, configure a header such as `RangeMode=UPDATE;RangeStartDate=2026-11-11` with the intended date. Write operations send JSON and return the employee response object. Explicit HTTP 429 responses are retried; connection failures are surfaced without automatic retry to avoid repeating a create accepted before the connection failed. Live writes have not been tested.

Imports read the response's `items` array and paginate with `limit` and zero-based `offset`. Page Size defaults to 25. Keep it at or below the maximum honored by your environment: NIM stops when a page contains fewer records than requested. No fixed page-count limit is imposed.

Oracle identifies Employees (`emps`) as a controlled availability resource planned for deprecation and recommends Workers for new integrations. Employees is included as requested and requires availability in your tenant.

References: [Oracle HCM REST API](https://docs.oracle.com/en/cloud/saas/human-resources/farws/index.html), [authentication and setup](https://docs.oracle.com/en/cloud/saas/human-resources/farws/Quick_Start.html), [Employees availability](https://docs.oracle.com/en/cloud/saas/human-resources/farws/api-employees.html), [Departments endpoint](https://docs.oracle.com/en/cloud/saas/human-resources/farws/op-organizations-get.html).

Operation references: [Create an employee](https://docs.oracle.com/en/cloud/saas/human-resources/farws/op-emps-post.html), [Update an employee](https://docs.oracle.com/en/cloud/saas/human-resources/farws/op-emps-empsuniqid-patch.html), [Get all assignments](https://docs.oracle.com/en/cloud/saas/human-resources/farws/op-emps-empsuniqid-child-assignments-get.html).
