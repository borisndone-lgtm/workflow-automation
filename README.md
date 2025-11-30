import os
import base64
import time
from email.mime.text import MIMEText
from googleapiclient.discovery import build
from google.oauth2.credentials import Credentials

# Authenticate with Gmail API using OAuth2
def authenticate_gmail():
    creds = Credentials.from_authorized_user_file('token.json')
    service = build('gmail', 'v1', credentials=creds)
    return service

# Read unread emails
def get_unread_emails(service):
    results = service.users().messages().list(userId='me', labelIds=['INBOX', 'UNREAD']).execute()
    messages = results.get('messages', [])
    return messages

# Send reply
def send_reply(service, to, subject, body):
    message = MIMEText(body)
    message['to'] = to
    message['subject'] = subject
    raw = base64.urlsafe_b64encode(message.as_bytes()).decode()
    message_send = service.users().messages().send(userId='me', body={'raw': raw}).execute()
    return message_send

# Workflow loop
def restaurant_auto_reply():
    service = authenticate_gmail()
    while True:
        messages = get_unread_emails(service)
        for msg in messages:
            msg_data = service.users().messages().get(userId='me', id=msg['id']).execute()
            headers = msg_data['payload']['headers']
            for header in headers:
                if header['name'] == 'From':
                    sender = header['value']
                    break
            # Customize reply based on subject/key words
            subject = "Thank you for contacting [Your Restaurant]"
            body = "Hello! Thank you for your email. We'll get back to you soon.\n\nBest regards,\n[Your Restaurant]"
            send_reply(service, sender, subject, body)
            # Mark as read
            service.users().messages().modify(userId='me', id=msg['id'], body={'removeLabelIds': ['UNREAD']}).execute()
        time.sleep(60)  # Run every 60 seconds

if __name__ == '__main__':
    restaurant_auto_reply()
