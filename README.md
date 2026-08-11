# salesforcecloud


<img width="1600" height="900" alt="Capture" src="https://github.com/user-attachments/assets/af365055-edfd-4115-9536-a9bf05b25539" />


code:

public with sharing class MailingService {

    /**
     * Preserves original method signature to fix your compilation error.
     * Safely checks for null parameters and handles exceptions.
     */
    public static void sendSimpleEmail(List<String> toAddresses, String subject, String body, Boolean isHtml) {
        if (toAddresses == null || toAddresses.isEmpty()) {
            System.debug('MailingService Error: No recipients specified.');
            return;
        }

        Messaging.SingleEmailMessage mail = new Messaging.SingleEmailMessage();
        mail.setToAddresses(toAddresses);
        mail.setSubject(subject);

        if (isHtml == true) {
            mail.setHtmlBody(body);
        } else {
            mail.setPlainTextBody(body);
        }

        // Executes via the unified list handler
        executeSend(new List<Messaging.SingleEmailMessage>{ mail });
    }

    /**
     * Preserves original template method signature.
     * Adds key null safety validations to prevent runtime null pointer exceptions.
     */
    public static void sendTemplateEmail(Id targetObjectId, Id templateId, Id whatId) {
        // Safety validation to prevent common runtime failures
        if (targetObjectId == null || templateId == null) {
            System.debug('MailingService Error: targetObjectId and templateId are required for template execution.');
            return;
        }

        Messaging.SingleEmailMessage mail = new Messaging.SingleEmailMessage();
        mail.setTargetObjectId(targetObjectId);
        mail.setTemplateId(templateId);
        
        if (whatId != null) {
            mail.setWhatId(whatId);
        }

        // Executes via the unified list handler
        executeSend(new List<Messaging.SingleEmailMessage>{ mail });
    }

    /**
     * Centralized execution method.
     * Uses partial success (allOrNone = false) so one bad email won't fail the whole transaction.
     */
    private static void executeSend(List<Messaging.SingleEmailMessage> emails) {
        if (emails == null || emails.isEmpty()) {
            return;
        }

        try {
            // Setting the second parameter to false enables partial success routing
            List<Messaging.SendEmailResult> results = Messaging.sendEmail(emails, false);
            inspectResults(results);
        } catch (Exception ex) {
            System.debug('MailingService critical error during send transaction: ' + ex.getMessage());
        }
    }

    /**
     * Loops over execution results arrays to handle and log structural errors.
     */
    private static void inspectResults(List<Messaging.SendEmailResult> results) {
        for (Messaging.SendEmailResult result : results) {
            if (result.isSuccess()) {
                System.debug('MailingService: Email dispatched successfully.');
            } else {
                for (Messaging.SendEmailError error : result.getErrors()) {
                    System.debug('MailingService Error: ' + error.getStatusCode() + ' - ' + error.getMessage());
                }
            }
        }
    }
}
