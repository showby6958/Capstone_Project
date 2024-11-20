// 캘린더 요구 설명서

## 데이터베이스 설계
calender_notes - 개인 캘린더의 날짜별 메모를 저장하는 테이블
shared_notes - 메모 공유 정보를 저장하는 테이블

보내는 사람(sender_id): 메모를 공유하는 사용자
받는 사람(recipient_id): 메모를 공유받는 사용자
메모 ID(note_id): 공유하려는 특정 메모의 ID

예시
-유저 A(ID: 1)가 유저 B(ID: 2)에게 자신의 메모(ID: 5)를 공유한다고 가정하면
sneder_id = 1 (유저 A의 ID)
recipient_id = 2 (유저 B의 ID)
note_id = 5 (유저 A가 만든 메모의 ID)


## API 엔드포인트 설계
- 메모 추가 기능 (유저가 날짜별로 메모를 추가할 수 있음)
- 메모 공유 기능 (특정 유저에게 메모를 공유할 수 있음)
- 공유된 메모 가져오기 기능 (공유된 메모를 다른 유저가 조회 가능)

### API 엔드포인트 설계
1. 캘린더에 메모 추가

app.post('/calendar/note', authenticateToken, (req, res) => {
    const { note_date, note_text } = req.body;
    const userId = req.user.userId;

    const sql = 'INSERT INTO calendar_notes (user_id, note_date, note_text) VALUES (?, ?, ?)';
    const values = [userId, note_date, note_text];

    db.query(sql, values, (error, result) => {
        if (error) {
            console.log(error);
            return res.status(500).json({ message: 'Error adding note' });
        }
        res.status(201).json({ message: 'Note added successfully' });
    });
});

- 호출

POST /calender/note

body:
{
    "note_date": "2024-11-12",
    "note_text": "Team meeting at 10 AM"
}


2. 메모 공유

app.get('/calendar/shared', authenticateToken, (req, res) => {
    const userId = req.user.userId;

    const sql = `
        SELECT 
            cn.note_date, 
            cn.note_text, 
            u.email AS shared_by 
        FROM shared_notes sn
        JOIN calendar_notes cn ON sn.note_id = cn.id
        JOIN users u ON sn.sender_id = u.id
        WHERE sn.recipient_id = ?
    `;

    db.query(sql, [userId], (error, results) => {
        if (error) {
            console.log(error);
            return res.status(500).json({ message: 'Error fetching shared notes' });
        }
        res.status(200).json(results);
    });
});

- 호출

POST /calender/share

body:
{
    "recipient_id": 2,
    "note_id": 1
}


3. 공유된 메모 조회





