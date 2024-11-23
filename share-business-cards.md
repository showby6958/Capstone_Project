## 명함 공유 엔드포인트 설계
### 1. 명함을 특정 사용자와 공유하는 엔드포인트
```
app.post('/business-card/share', authenticateToken, (req, res) => {
  const { recipient_id, business_card_id } = req.body;
  const sender_id = req.user.user_id;

  const sql = `INSERT INTO shared_business_cards (sender_id, recipient_id, business_card_id) VALUES (?, ?, ?)`;
  const values = [sender_id, recipient_id, business_card_id];

  db.query(sql, values, (error, result) => {
    if (error) {
      console.log(error);
      res.status(500).json({message: 'Error sharing business card'});
    }
    res.status(201).json({message: 'Business card shared successfully'});
  });
});
```

### 2. 공유받은 명함을 조회하는 엔드포인트
```
app.get('/business-card/shared', authenticateToken, (req, res) => {
  const userId = req.user.userId;

  const sql = `
    SELECT 
      bc.name,
      bc.title,
      bc.phone,
      bc.email,
      u.email AS shared_by
    FROM shared_business_cards sbc
    JOIN business_cards ON sbc.business_card_id = bc.id
    JOIN users u ON sbc.sender_id = u_id
    WHERE sbc.recipient_id = ?
    `;

    db.query(sql, [userId], (error, result) => {
      if(error) {
        console.error(error);
        return res.status(500).json({message: 'Error fetching shared business cards'});
      }
      res.status(200).json(result);
    });
});
```
### 설계 명세서
## 1. 명함 공유
- URL : POST http://localhost:3000/business-card/share
- Headers : Authorization: Bearer <JWT 토큰> (공유하고자 하는 유저의 JWT 토큰)
- Body : 
```
{
    "recipient_id": 2,
    "business_card_id": 1
}
```
- 응답 : 
```
{
    "message": "Business card shared successfully"
}
```

## 2. 공유받은 명함 조회
- URL : GET http://localhost:3000/business-card/shared
- Headers : Authorization : Bearer <JWT 토큰> (공유 받은 유저(recipient_id)의 JWT 토큰)
- 응답 : 
```
[
    {
        "name": "John Doe",
        "title": "Software Engineer",
        "company": "Tech Company",
        "phone": "123-456-7890",
        "email": "johndoe@example.com",
        "shared_by": "alice@example.com"
    }
]
```
