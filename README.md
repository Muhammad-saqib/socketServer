Here’s an update from my side for today: I’ve added test cases related to the search functionality using Cucumber. This approach has been particularly helpful in achieving the Gherkin style, making it easier to write and understand test cases. It also enables Imisi to contribute by writing test cases and cross-verifying them. I’ll be giving a demo today, and we can discuss it further during the session.

If needed, I was discussing this with James, who raised an excellent point that I believe is valuable for our ongoing discussion. I’m including it in this email to ensure we don’t overlook it. In the manual sheet, there are several test cases that don’t need to be part of the integration tests. We need to clearly identify which test cases are required for integration testing.

From the feature sheets, I can easily spot test cases that we can skip for integration testing and instead include in functional tests. However, I think there are a few test cases that are necessary for integration testing.

I am adding another column to the sheet to indicate whether integration test cases are required for each item. This will help us easily identify and cover the necessary test cases.

James also raised a very valid point. Let’s take the 409 scenario cases as an example. The DB Phoenix team needs to add tests to ensure the response remains consistent. If they had already included a test case to verify that the ticket contains all the required fields returned by the server, we wouldn’t have faced issues, and the effort required would have been minimal.

For instance, we previously encountered a problem where a script was missing, and the response did not include a valid ticket type. Although we use mock responses and assume that Phoenix returns everything as specified, if they fail to return the required data, they will need to fix it. In such cases, we don’t necessarily need to write integration test cases. Phoenix needs to ensure that whatever has been discussed is returned as expected.

While we can write integration test cases on our end to confirm that the response aligns with our SnapFlo requirements, it would involve additional effort. However, doing so would ensure that everything is consistent and reliable.

Our purpose in integration testing should be to write the minimum set of test cases required to verify that the responses provided by other teams align with the agreed-upon specifications. These test cases will validate the responses in both RM-API and RM-UI. This approach ensures that any discrepancies or issues are identified early, enabling the other teams to address and resolve them promptly. By focusing on verifying agreed responses, we streamline our efforts while maintaining accountability for fixes on their end if anything goes wrong.
