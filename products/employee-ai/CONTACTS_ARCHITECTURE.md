# Contacts Architecture — DBL Employee AI

هذه وثيقة مرجعية معمارية لContacts داخل DBL Employee AI.

## المبادئ

- Contact هوية عامة مستقلة عن provider.
- Channel Identity كيان منفصل.
- WhatsApp هو القناة المدعومة حاليًا، لكن المعمارية channel-ready.
- Contact ID داخلي UUID وليس phone/email/provider ID.
- WhatsApp account scope يعتمد على receiving `phone_number_id` snapshot.
- لا automatic merge ولا delete للContacts في MVP.
- Browser writes إلى Contacts تظل fail-closed؛ ingestion/server side هو الموثوق.

## ملاحظة

للتفاصيل التاريخية الدقيقة راجع Git history والنسخة السابقة من Product Memory قبل إعادة الهيكلة.
