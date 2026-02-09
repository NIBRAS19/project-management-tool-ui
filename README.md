📅 How to Build an ICS Calendar Email System
A Super Simple Guide

🎯 What Are We Building?
Imagine you want to invite your friend to a birthday party. You could:

Tell them the date and time (they might forget!)
Send them a magic calendar card that automatically adds the party to their phone calendar! ✨
That "magic calendar card" is called an ICS file. This guide teaches you how to send these magic cards through email!

🧩 The 3 Pieces You Need
Think of it like making a sandwich:

Piece	What It Does	Like...
1. ICS Generator	Creates the calendar card	The bread
2. Email Service	Sends the email	The plate
3. Email Template	Makes it look pretty	The decoration
📝 Piece 1: The ICS Generator
What is an ICS File?
An ICS file is just a text file that calendars understand. It looks like this:

BEGIN:VCALENDAR
VERSION:2.0
BEGIN:VEVENT
SUMMARY:Birthday Party
DTSTART:20260210T140000
DTEND:20260210T160000
UID:unique-id-123@mywebsite.com
END:VEVENT
END:VCALENDAR
The Magic Words Explained
Word	Meaning	Example
BEGIN:VCALENDAR	"Hey calendar, listen up!"	-
VERSION:2.0	"I'm speaking calendar language version 2"	-
BEGIN:VEVENT	"Here comes an event..."	-
SUMMARY	The event's name	Birthday Party
DTSTART	When it starts	Feb 10, 2026 at 2:00 PM
DTEND	When it ends	Feb 10, 2026 at 4:00 PM
UID	A unique ID (like a fingerprint)	unique-id-123@mywebsite.com
END:VEVENT	"Event info complete!"	-
END:VCALENDAR	"All done, calendar!"	-
PHP Code to Generate ICS
Create a file called IcsGenerator.php:

<?php
class IcsGenerator {
    
    private $organizerEmail;
    private $organizerName;
    
    public function __construct($email, $name) {
        $this->organizerEmail = $email;
        $this->organizerName = $name;
    }
    
    /**
     * Create a new calendar event
     */
    public function createEvent($eventData) {
        // Generate unique ID (like a fingerprint for this event)
        $uid = 'event-' . $eventData['id'] . '@mywebsite.com';
        
        // Format dates for calendar (YYYYMMDDTHHMMSS)
        $startDate = date('Ymd', strtotime($eventData['date']));
        $startTime = str_replace(':', '', $eventData['start_time']);
        $endTime = str_replace(':', '', $eventData['end_time']);
        
        // Build the ICS content
        $ics = "BEGIN:VCALENDAR\r\n";
        $ics .= "VERSION:2.0\r\n";
        $ics .= "PRODID:-//My App//Calendar//EN\r\n";
        $ics .= "METHOD:REQUEST\r\n";
        $ics .= "BEGIN:VEVENT\r\n";
        $ics .= "UID:{$uid}\r\n";
        $ics .= "DTSTAMP:" . gmdate('Ymd\THis\Z') . "\r\n";
        $ics .= "DTSTART:{$startDate}T{$startTime}\r\n";
        $ics .= "DTEND:{$startDate}T{$endTime}\r\n";
        $ics .= "SUMMARY:{$eventData['title']}\r\n";
        $ics .= "DESCRIPTION:{$eventData['description']}\r\n";
        $ics .= "ORGANIZER;CN={$this->organizerName}:mailto:{$this->organizerEmail}\r\n";
        $ics .= "SEQUENCE:0\r\n";
        $ics .= "STATUS:CONFIRMED\r\n";
        $ics .= "END:VEVENT\r\n";
        $ics .= "END:VCALENDAR\r\n";
        
        return $ics;
    }
    
    /**
     * Update an existing event (increment SEQUENCE!)
     */
    public function updateEvent($eventData, $sequence = 1) {
        // Same as create, but with higher SEQUENCE number
        $uid = 'event-' . $eventData['id'] . '@mywebsite.com';
        
        $startDate = date('Ymd', strtotime($eventData['date']));
        $startTime = str_replace(':', '', $eventData['start_time']);
        $endTime = str_replace(':', '', $eventData['end_time']);
        
        $ics = "BEGIN:VCALENDAR\r\n";
        $ics .= "VERSION:2.0\r\n";
        $ics .= "PRODID:-//My App//Calendar//EN\r\n";
        $ics .= "METHOD:REQUEST\r\n";
        $ics .= "BEGIN:VEVENT\r\n";
        $ics .= "UID:{$uid}\r\n";  // SAME UID as original!
        $ics .= "DTSTAMP:" . gmdate('Ymd\THis\Z') . "\r\n";
        $ics .= "DTSTART:{$startDate}T{$startTime}\r\n";
        $ics .= "DTEND:{$startDate}T{$endTime}\r\n";
        $ics .= "SUMMARY:{$eventData['title']}\r\n";
        $ics .= "SEQUENCE:{$sequence}\r\n";  // HIGHER than before!
        $ics .= "STATUS:CONFIRMED\r\n";
        $ics .= "END:VEVENT\r\n";
        $ics .= "END:VCALENDAR\r\n";
        
        return $ics;
    }
    
    /**
     * Cancel an event
     */
    public function cancelEvent($eventData, $sequence = 1) {
        $uid = 'event-' . $eventData['id'] . '@mywebsite.com';
        
        $startDate = date('Ymd', strtotime($eventData['date']));
        $startTime = str_replace(':', '', $eventData['start_time']);
        $endTime = str_replace(':', '', $eventData['end_time']);
        
        $ics = "BEGIN:VCALENDAR\r\n";
        $ics .= "VERSION:2.0\r\n";
        $ics .= "PRODID:-//My App//Calendar//EN\r\n";
        $ics .= "METHOD:CANCEL\r\n";  // Changed to CANCEL!
        $ics .= "BEGIN:VEVENT\r\n";
        $ics .= "UID:{$uid}\r\n";
        $ics .= "DTSTAMP:" . gmdate('Ymd\THis\Z') . "\r\n";
        $ics .= "DTSTART:{$startDate}T{$startTime}\r\n";
        $ics .= "DTEND:{$startDate}T{$endTime}\r\n";
        $ics .= "SUMMARY:CANCELLED: {$eventData['title']}\r\n";
        $ics .= "SEQUENCE:{$sequence}\r\n";
        $ics .= "STATUS:CANCELLED\r\n";  // Mark as cancelled!
        $ics .= "END:VEVENT\r\n";
        $ics .= "END:VCALENDAR\r\n";
        
        return $ics;
    }
}
?>
🔑 The Golden Rules
Rule	Why It Matters
UID must be unique	This is how calendars know which event to update/delete
UID must stay the same	When updating, use the SAME UID so it updates instead of creating a new event
SEQUENCE must increase	Higher number = newer version. Calendar keeps the highest one
METHOD:REQUEST = Create/Update	Tells calendar "please add/update this"
METHOD:CANCEL = Delete	Tells calendar "please remove this"
📧 Piece 2: The Email Service
Now we need to send the ICS file attached to an email. We'll use PHPMailer.

Step 1: Install PHPMailer
composer require phpmailer/phpmailer
Step 2: Create EmailService.php
<?php
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

require 'vendor/autoload.php';
require 'IcsGenerator.php';

class EmailService {
    
    private $mailer;
    private $icsGenerator;
    
    public function __construct() {
        $this->mailer = new PHPMailer(true);
        $this->icsGenerator = new IcsGenerator(
            'your-email@gmail.com',
            'Your App Name'
        );
        
        // Configure SMTP
        $this->mailer->isSMTP();
        $this->mailer->Host = 'smtp.gmail.com';
        $this->mailer->SMTPAuth = true;
        $this->mailer->Username = 'your-email@gmail.com';
        $this->mailer->Password = 'your-app-password';  // See note below!
        $this->mailer->SMTPSecure = 'tls';
        $this->mailer->Port = 587;
    }
    
    /**
     * Send a new event invitation
     */
    public function sendNewEvent($toEmail, $eventData) {
        // Generate the ICS content
        $icsContent = $this->icsGenerator->createEvent($eventData);
        
        // Send the email
        return $this->sendEmail(
            $toEmail,
            'New Event: ' . $eventData['title'],
            'You have been invited to: ' . $eventData['title'],
            $icsContent,
            'invite.ics'
        );
    }
    
    /**
     * Send an event update
     */
    public function sendEventUpdate($toEmail, $eventData, $sequence) {
        $icsContent = $this->icsGenerator->updateEvent($eventData, $sequence);
        
        return $this->sendEmail(
            $toEmail,
            'Event Updated: ' . $eventData['title'],
            'The event has been updated.',
            $icsContent,
            'update.ics'
        );
    }
    
    /**
     * Send an event cancellation
     */
    public function sendEventCancellation($toEmail, $eventData, $sequence) {
        $icsContent = $this->icsGenerator->cancelEvent($eventData, $sequence);
        
        return $this->sendEmail(
            $toEmail,
            'Event Cancelled: ' . $eventData['title'],
            'The event has been cancelled.',
            $icsContent,
            'cancel.ics'
        );
    }
    
    /**
     * The actual email sending function
     */
    private function sendEmail($to, $subject, $body, $icsContent, $icsFilename) {
        try {
            $this->mailer->clearAddresses();
            $this->mailer->clearAttachments();
            
            $this->mailer->setFrom('your-email@gmail.com', 'Your App');
            $this->mailer->addAddress($to);
            
            $this->mailer->isHTML(true);
            $this->mailer->Subject = $subject;
            $this->mailer->Body = $body;
            
            // THE MAGIC: Attach the ICS file!
            $this->mailer->addStringAttachment(
                $icsContent,                    // The ICS text
                $icsFilename,                   // Filename (invite.ics)
                'base64',                       // Encoding
                'text/calendar; method=REQUEST' // MIME type
            );
            
            $this->mailer->send();
            return ['success' => true, 'message' => 'Email sent!'];
            
        } catch (Exception $e) {
            return ['success' => false, 'message' => $e->getMessage()];
        }
    }
}
?>
🔐 Gmail App Password
Gmail requires a special "App Password" instead of your regular password:

Go to Google Account Security
Enable 2-Step Verification
Click "App passwords"
Generate a new password for "Mail"
Use that 16-character password in your code
🎨 Piece 3: The Email Template (Optional)
Make your emails look pretty! Create EmailTemplate.php:

<?php
class EmailTemplate {
    
    public static function getEventEmail($eventData, $type = 'new') {
        $colors = [
            'new' => '#10B981',    // Green
            'update' => '#F59E0B', // Orange
            'cancel' => '#EF4444'  // Red
        ];
        
        $titles = [
            'new' => 'New Event',
            'update' => 'Event Updated',
            'cancel' => 'Event Cancelled'
        ];
        
        $color = $colors[$type];
        $title = $titles[$type];
        $date = date('l, F j, Y', strtotime($eventData['date']));
        $time = date('g:i A', strtotime($eventData['start_time']));
        
        return "
        <html>
        <body style='font-family: Arial, sans-serif; padding: 20px;'>
            <div style='background: {$color}; color: white; padding: 20px; border-radius: 8px;'>
                <h2>{$title}</h2>
            </div>
            <div style='padding: 20px; background: #f5f5f5;'>
                <p><strong>Event:</strong> {$eventData['title']}</p>
                <p><strong>Date:</strong> {$date}</p>
                <p><strong>Time:</strong> {$time}</p>
            </div>
            <p style='color: gray; font-size: 12px;'>
                Open the attached .ics file to add this to your calendar.
            </p>
        </body>
        </html>
        ";
    }
}
?>
🔧 Putting It All Together
Complete Example
<?php
require 'EmailService.php';

// 1. Create your event data
$event = [
    'id' => 123,
    'title' => 'Team Meeting',
    'description' => 'Weekly team sync',
    'date' => '2026-02-15',
    'start_time' => '10:00:00',
    'end_time' => '11:00:00'
];

// 2. Create the email service
$emailService = new EmailService();

// 3. Send the invitation!
$result = $emailService->sendNewEvent('friend@email.com', $event);

if ($result['success']) {
    echo "Email sent successfully!";
} else {
    echo "Error: " . $result['message'];
}
?>
🗄️ Database: Remember the Sequence!
To update/cancel events properly, you need to track:

Column	Type	Purpose
calendar_uid	VARCHAR(255)	The unique event ID
ics_sequence	INT	Version number (starts at 0)
Add to your events table:

ALTER TABLE events ADD COLUMN calendar_uid VARCHAR(255);
ALTER TABLE events ADD COLUMN ics_sequence INT DEFAULT 0;
When creating an event:

$uid = 'event-' . $eventId . '@mywebsite.com';
// Save $uid to calendar_uid column
// Save 0 to ics_sequence column
When updating:

$newSequence = $currentSequence + 1;
// Use $newSequence in the ICS file
// Save $newSequence to database
📱 How It Looks to the Recipient
Gmail: Shows a calendar preview at the top of the email with Yes/No/Maybe buttons
Outlook: Opens the ICS file and shows "Accept/Decline" options
Apple Mail: Shows "Add to Calendar" button
All of them: The attachment appears at the bottom
🎉 Summary: The Complete Flow
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Your App  │ --> │  ICS File   │ --> │   Email     │
│  (creates   │     │  (magic     │     │   (sends    │
│   event)    │     │   calendar  │     │   to user)  │
│             │     │   card)     │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
                                              │
                                              ▼
                                    ┌─────────────────┐
                                    │  User's Email   │
                                    │  (Gmail/Outlook)│
                                    │                 │
                                    │  Opens .ics     │
                                    │       ▼         │
                                    │  Event added to │
                                    │  their calendar!│
                                    └─────────────────┘
✅ Checklist for Your Project
[ ] Install PHPMailer (composer require phpmailer/phpmailer)
[ ] Create IcsGenerator.php
[ ] Create EmailService.php
[ ] Configure SMTP settings (Gmail App Password)
[ ] Add calendar_uid and ics_sequence columns to your database
[ ] Test: Send a new event
[ ] Test: Update the event (sequence should increase)
[ ] Test: Cancel the event
🆘 Troubleshooting
Problem	Solution
Email not sending	Check SMTP credentials and App Password
Event not updating	Make sure UID is the same and SEQUENCE is higher
Weird characters in calendar	Use \r\n for line breaks, not just \n
Event creates duplicate	The UID changed! Keep it consistent
That's it! You now know how to send magic calendar cards through email! 
