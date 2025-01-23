# Embedded models

Parts of non-embedded models (are accessable from a single context \ are associated only with a specific instance of another model)

#### **File Model** (Embedded)

```json
{
  "id": "UUID",
  "file_path": "str"
}
```

#### **Education Model** (Embedded)

```json
{
  "id": "UUID",
  "country": "str",
  "settlement": "str",
  "institution": "str",
  "faculty": "str",
  "speciality": "str",
  "from_date": "UNIX timestamp",
  "to_date": "UNIX timestamp",
  "attachments": ["File Model embedded"]
}
```

#### **Certificate Model** (Embedded)

```json
{
  "id": "UUID",
  "class": "int",
  "title": "str",
  "from_date": "UNIX timestamp",
  "to_date": "UNIX timestamp",
  "attachments": ["File Model embedded"]
}
```

#### **Expertise Model** (Embedded)

```json
{
  "id": "UUID",
  "organization": "str",
  "position": "str",
  "employment_type": "str",
  "work_type": "str",
  "country": "str",
  "from_date": "UNIX timestamp",
  "to_date": "UNIX timestamp",
  "attachments": ["File Model"]
}
```

#### **Wallet Model** (Tokenized \ Encrypted, Embedded)

```json
{
  "id": "UUID",
  "user_id": "UUID",
  "number_obfuscated": "str (a few characters replaced with '*')",
  "number_encrypted": "str",
  "name": "str",
  "expiration": "str",
  "code": "str"
}
```

#### **Address Model** (Tokenized \ Encrypted, Embedded)

```json
{
  "id": "UUID",
  "country": "str",
  "region": "str",
  "settlement": "str",
  "street": "str",
  "house": "str",
  "entrance": "str",
  "apartment_number": "str"
}
```

#### **MedCard Model** (Tokenized \ Encrypted, Embedded)

```json
{
  "id": "UUID",
  "chronic_diseases": ["str"],
  "allergies": ["str"],
  "prescribed_drugs": ["str"],
  "ambulatory_treatment": ["str"]
}
```

#### **Service Model**

```json
{
  "id": "UUID",
  "title": "str",
  "description": "str",
  "price": "float",
  "currency": "str",
  "duration": "int (seconds)"
}
```

# Non-embedded models

Independent models (can be accessed from different context \ are not associated with a specific instance of another model)

#### **User Model**

```json
{
  "id": "UUID",
  "type": "int",
  "name": "str",
  "surname": "str",
  "avatar": "File Model",
  "iin": "str (encrypted)",
  "sex": "str",
  "email": "str (encrypted)",
  "password": "str (hashed)",
  "phone_number": "str (encrypted)",
  "addresses": ["Address Model embedded (encrypted)"],
  "medicine_card": "MedCard Model embedded (encrypted)",
  "family": "str",
  "languages": "int (byte encoding)",
  "services": ["Service Model"],
  "currency": "str",
  
  "_location": "Address Model embedded",
  "_target_social_group": "Social Group Model embedded",
  "_expertise_years": "int",
  "_expertise_months": "int",
  "_educations": ["Education Model embedded"],
  "_expertise": ["Expertise Model embedded"],
  "_certificates": ["Certificate Model embedded"]
}
```

#### **Promos Model**

```json
{
  "id": "UUID",
  "code": "str",
  "discount": "float",
  "criteria": "str"
}
```

#### **Review Model**

```json
{
  "id": "UUID",
  "from_user_id": "UUID",
  "to_user_id": "UUID",
  "rating": "float",
  "comment": "str"
}
```

#### **Interaction Model** (Tokenized \ Encrypted)

```json
{
  "id": "UUID",
  "from_user_id": "UUID (encrypted)",
  "to_user_id": "UUID (encrypted)",
  "date": "UNIX timestamp (encrypted)",
  "directions": "str (encrypted)",
  "feedback": "str (encrypted)"
}
```

#### **Message Model**

```json
{
  "id": "UUID",
  "from_user_id": "UUID (encrypted)",
  "to_user_id": "UUID (encrypted)",
  "date_published": "int (unix) (encrypted)",
  "date_edited": "int (unix) (encrypted)",
  "content": "str (encrypted)",
  "attachment": "File Model (encrypted)"
}
```

#### **Session Model**

```json
{
  "id": "UUID",
  "from_user_id": "UUID",
  "to_user_id": "UUID (encrypted)",
  "session_class": "int",
  "service": "Service Model",
  "from": "int (seconds)",
  "to": "int (seconds)",
  "location": "Address model",
  "break": "int (seconds)",
  "date": "int (unix)",
  "repeat": "int (binary)"
}
```

#### **Payment Model**

```json
{
  "id": "UUID",
  "date": "UNIX timestamp (encrypted)",
  "from_user_id": "UUID (encrypted)",
  "to_user_id": "UUID (encrypted)",
  "amount": "float (encrypted)",
  "cheque": "str (encrypted)"
}
```

