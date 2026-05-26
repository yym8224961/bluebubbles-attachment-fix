# BlueBubbles Attachment API Fix

**Bug**: koa-body/formidable multipart parser truncates form field values containing `;`. iMessage chat GUIDs like `iMessage;-;<email_or_phone>` become `iMessage`, causing attachments to route to the wrong chat.

**Fix**: Modified `sendAttachment` handler to fall back to URL query parameter when multipart body value is truncated.

**Environment**: macOS 15.7.3 | BlueBubbles Server 1.9.9 | Private API enabled | Intel iMac

## Quick Apply

Extract BB asar, apply the patch, repack, restart:
```bash
cd /Applications/BlueBubbles.app/Contents/Resources
cp app.asar app.asar.bak
npx asar extract app.asar /tmp/bb-source
# Apply dist-main.patch to /tmp/bb-source/dist/main.js
npx asar pack /tmp/bb-source app.asar
killall BlueBubbles && open /Applications/BlueBubbles.app
```

## Usage After Fix

Send attachments with chatGuid in URL query:
```bash
curl -X POST \
  "http://BB_HOST:1234/api/v1/message/attachment?password=PASS&chatGuid=iMessage;-;<guid>" \
  -F "chatGuid=iMessage;-;<guid>" \
  -F "name=file.pdf" \
  -F "attachment=@/path/to/file"
```

## Native Voice Messages (iMessage)

With Private API + isAudioMessage:
```bash
curl -X POST \
  "http://BB_HOST:1234/api/v1/message/attachment?password=PASS&chatGuid=..." \
  -F "chatGuid=..." \
  -F "method=private-api" \
  -F "isAudioMessage=true" \
  -F "attachment=@/voice.wav" \
  -F "name=voice.wav"
```

## Related

- Original issue: [#804](https://github.com/BlueBubblesApp/bluebubbles-server/issues/804)
- Investigation: SIP / AMFI / Launch Constraints / DYLIB / multipart parser
