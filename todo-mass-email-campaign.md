# Club Sixty Six - Mass Member Email Campaign System

## Purpose
Send personalized, individual emails to all 26 migrated members with billing setup instructions.

## Requirements

### Template System
- [ ] Create HTML email template with Olivia's branding
- [ ] Personalize with member name, email
- [ ] Track send status per member
- [ ] Prevent duplicate sends

### Email Features
- [ ] Individual "To" field (not BCC) - each member gets their own email
- [ ] Friendly unsubscribe link (one-click)
- [ ] "Complete Setup" button deep-links to account → billing
- [ ] Responsive design for mobile

### Tracking
- [ ] Send list: track which of 26 members received email
- [ ] Retry failed sends
- [ ] Bounce handling

### Integration Option 1: Resend (Already in use)
- Uses existing Resend API integration
- Simple, already authenticated
- Good for transactional emails

### Integration Option 2: n8n workflow
- Visual workflow builder
- Can pull from DB, loop through members
- Track status in Notion/Airtable

## Member List (to send to)
The 26 ACTIVE members from the migration:
1. wayne.ledrew@tech-method.com
2. jasmine@ledrew.org
3. amberly.pye@icloud.com
4. elaineler.1107@gmail.com
5. meda4worthy@gmail.com
6. alysha.raposo@gmail.com
7. asiddall_2000@yahoo.ca
8. gfortune@nancycampbell.ca
9. lindsay.egan83@gmail.com
10. shellyweber14@gmail.com
11. triciajaneen@gmail.com
12. hailes812@gmail.com
13. kvendrig1@gmail.com
14. annelavinia@hotmail.com
15. isabelle.branscombe@gmail.com
16. amandalimm2@gmail.com
17. ljmorganelli@gmail.com
18. tonialberto13@gmail.com
19. dmlutz@uwaterloo.ca
20. graceortlieb@hotmail.com
21. danielleortlieb@outlook.com
22. medmeades5@gmail.com
23. halleyjeffs97@hotmail.com
24. danielle.dobbs03@gmail.com
25. mckerrowq@gmail.com
26. wayne-test@clubsixty.ca (exclude - test account)

## Next Steps
1. Olivia reviews email template
2. Choose integration method (Resend script vs n8n)
3. Schedule sends (staggered to avoid spam filters?)

## Notes
- MUST get Olivia approval before sending
- Include clear deadline for setup completion
- Provide support contact for issues
