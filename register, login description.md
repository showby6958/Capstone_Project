## 회원가입
```
// 회원가입 엔드포인트 (비밀번호를 bcryptjs로 해싱해서 사용자 정보를 DB에 저장한다.)
app.post('/register', async (req, res) => {
  const { email, password } = req.body;

  try {
    // 이메일 중복 체크
    db.query('SELECT * FROM users WHERE email = ?', [email], async (error, result) => {
      if (result.length > 0) {
        return res.status(400).json({message: 'User already exist'});
      }

      // 비밀번호 해싱 후 저장
      const hashedPassword = await bcrypt.hash(password, 10);
      db.query('INSERT INTO users (email, password) VALUES (?, ?)', [email, hashedPassword], (error) => {
        if (error) throw error;
        res.status(201).json({message: 'User registered successfully'});
      });
    });
  } catch (error) {
    res.status(500).json({message: 'Error registering user'});
  }
});
```

## 로그인 
```
// 로그인 기능 구현 (JWT를 생성해서 로그인한 사용자에게 토큰을 발급한다.)
app.post('/login', (req, res) => {
  const { email, password } = req.body;

  try {
    db.query('SELECT * FROM users WHERE email = ?', [email], async (error, result) => {
      if (result.length === 0) {
        return res.status(400).json({message: 'Invalid email or password'});
      }

      const user = result[0];
      // bcrypt.compare() 함수는 사용자가 입력한 평문 비밀번호(password)와 데이터베이스에 저장된 해시된 비밀번호(user.password)를 비교 일치하면 true 아니면 false
      const isPasswordValid = await bcrypt.compare(password, user.password); 
      if (!isPasswordValid) {
        return res.status(400).json({message: 'Invalid email or password'});
      }

      // jwt.sign() 함수는 사용자 ID(userId)를 페이로드로 갖는 토큰을 생성하고, 토큰의 만료 시간을 24시간(expiresIn: '24h')으로 설정합니다. 
      // SECRET_KEY는 토큰의 서명을 확인할 때 사용됩니다.
      const token = jwt.sign({ userId: user.id }, SECRET_KEY, { expiresIn: '24h' });
      res.json({ token }); // 생성된 JWT 토큰을 클라이언트에 반환(클라이언트가 이 토큰을 받아 이후 요청에서 인증에 사용할 수 있도록 한다.)
    });
  } catch (error) {
    res.status(500).json({message: 'Error logging in'});
  }
});
```

## 인증된 경로 설정 예시
```
// 인증된 경로 설정 예시
app.get('/protected', authenticateToken, (req, res) => {
  res.json({message: 'You have accessed a protected route!'});
});
```

### 설계 명세서
## 1. 회원가입
- URL : POST http://localhost:3000/register
- Body : 
```
{
  "email": "example@gmail.com",
  "password": "example"
}
```
- 응답 : 
```
{
    "message": "User registered successfully"
}
```
- 응답(이메일 중복 체크) :
```
{
    "message": "User already exist"
}
```
- 응답(오류) :
```
{
    "message": "Error registering user"
}
```

## 2. 로그인
- URL : POST http://localhost:3000/login
- Body : 
```
{
  "email": "example@gmail.com",
  "password": "example"
}
```
- 응답 : 
```
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQsImlhdCI6MTczMjcyMjA0OSwiZXhwIjoxNzMyODA4NDQ5fQ.71XLyp0ggM1wdb7_QWJRBsi_a9uaaDA1v5ibPrwFdxo"
}
```

- 응답(로그인 실패) :
```
{
    "message": "Invalid email or password"
}
```

- 응답(오류) :
```
{
    "message": "Error logging in"
}
```
