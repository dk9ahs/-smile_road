# SpringBoot-Project-SmileRoad 
스프링 부트 + JSP 교통약자편의시설 위치 공유 사이트

## 프로젝트 소개
교통 약자들가 필요로 하는 편의시설의 위치 정보를 제공해 생활 범위 확장, 외출시 현실적인 어려움과 불편함을 해소시키고자 웹사이트를 기획하였습니다.

## 개발 기간 
2024.7.31 ~ 2024.09.05

### 개발 환경
- `Java 17`
- `JDK 17.0.11`
- **IDE** : STS 4.0
- **Framework** : Springboot(3.3.2)
- **Database** : Oracle DB(21c)
- **ORM** : Mybatis

## 담당 기능
- Firebase Realtime을 이용한 실시간 채팅 기능 구현
    - 이용자와 관리자가 1:1 실시간 채팅할 수 있도록 채팅방 기능을 구현했습니다.
    - 이용자는 즉시 관리자와 연결될 수 있으며, 관리자는 새로운 문의가 도착할 때 채팅방 리스트에서 선택하여 채팅을 시작할 수 있습니다.
    - 메시지 전송과 채팅방 삭제는 Ajax를 통해 서버로 비동기 전송하여 실시간으로 반영되도록 처리했습니다.
    - 채팅방 리스트 조회는 CompletableFuture를 활용해 비동기 방식으로 목록을 반환하여 빠른 응답을 제공했습니다.
 
    <details>
            <summary>ChatController</summary>
        
        chatService.getMessagesByChatRoomId(chatRoomId)
        	            .thenApply(messages -> {
        	                model.addAttribute("messages", messages);
        	                re@Controller
        public class ChatController {
        	
        	@Autowired
            private ChatService chatService;
        	
        	// 채팅방 전 확인 팝업
        	@RequestMapping("member/chatPopup")
        	public String chatPopup(Model model) {
        		String sId = SecurityContextHolder.getContext().getAuthentication().getName();
        		
        		model.addAttribute("Id", sId);
        		return "member/chatPopup";
        	}
        	
        	// 채팅방
        	@RequestMapping("member/chatRoom")
        	public CompletableFuture<Object> chatRoom(HttpServletRequest request, Model model)
        	{
        		String sId = SecurityContextHolder.getContext().getAuthentication().getName();
        		String chatRoomId = request.getParameter("chatRoomId");
        	    model.addAttribute("Id", sId); // 로그인된 아이디
        	    model.addAttribute("chatRoomId", chatRoomId);
        	    
        	    return turn "member/chatRoom";
        	            });
        	}
        	
        	// 메세지 보내는 기능
        	@PostMapping("member/chatRoom/send")
        	public ResponseEntity<MessageDto> sendMessage(@RequestBody MessageDto message) {
        		chatService.sendMessage(message);
        	    return ResponseEntity.ok(message);
        	}
        	
        	// 관리자 전용 채팅방리스트
        	@GetMapping("admin/chatRoomList")
        	public CompletableFuture<String> chatRoomList(Model model) {
        	    return chatService.getChatRoomList()
        	        .thenApply(chatRooms -> {
        	            model.addAttribute("chatRooms", chatRooms);
        	            return "admin/chatRoomList"; // chatRoomList.jsp 페이지로 이동
        	        });
        	}
        	
        	 @DeleteMapping("member/chatRoom/deleteChatRoom")
        	    public ResponseEntity<String> deleteChatRoom(HttpServletRequest request) {
        	        try {
        		    		String chatRoomId = request.getParameter("chatRoomId");
        		            chatService.deleteChatRoom(chatRoomId);
        		            
        	            return ResponseEntity.ok("채팅방이 성공적으로 삭제되었습니다.");
        	        } catch (Exception e) {
        	            return ResponseEntity.status(500).body("채팅방 삭제에 실패했습니다: " + e.getMessage());
        	        }
        	    }
        }
    </details>
    
- Oracle 기반 계층형 구조와 MyBatis를 활용한 문의 게시판 CRUD 구현
    - Spring Security을 이용해 일반 사용자는 문의글을 작성하고, 관리자는 답변을 작성할 수 있도록 권한을 설정하였습니다.
    - 문의 게시판의 CRUD 기능과 비밀글 기능을 구현하여 사용자별 접근 권한을 강화했습니다.
    - 비밀번호 변경은 AJAX를 이용해 비동기 방식으로 서버에 요청하며 변경 완료시 입력란과 버튼을 비활성화 하여 변경 완료 상태를 시각적으로 전달하였습니다.


## 주요기능
#### 로그인 
- DB값 검증
- 네이버 SMTP - 이메일 전송, Redis - 이메일,인증번호 저장하여 ID찾기, PW찾기
- 중복 로그인 방지
- 로그인 시 쿠키(Cookie) 및 세션(Session) 생성
- 로그인 실패시 계정 잠금, 오류메세지 구현

#### 회원가입
- 주소 API 연동
- ID 중복 체크
- 소셜 로그인 (카카오, 네이버, 구글)

#### 마이 페이지
- 주소 API 연동
- 회원 정보 변경
- PW 변경
- 회원 탈퇴

#### 위치 정보
- DB 연동과 범위 지정
- 현재 위치 정보 제공
- 카카오 지도 API 연동 (마커, 축소및 확대, 지도 타입 변경)
- 위치 정보 모달창 구현

#### 공지사항 게시판, 커뮤니티 게시판
- 글 작성, 읽기, 수정, 삭제(CRUD)
- 새 글, 파일 첨부 시 뱃지
- 페이징 및 검색
- 댓글

#### 문의사항 게시판
- 글 작성, 읽기, 수정, 삭제(CRUD)
- 비밀글 구현
- 관리자일시 답글 작성

#### 채팅 상담
- Firebase 연동
- 팝업으로 채팅 입장 확인
- 메세지 전송
- 채팅방 삭제
- 관리자일시 채팅방 리스트

#### 관리자 페이지
- 회원 목록 및 상세정보 확인
- 회원 권한 및 잠금 설정
- 회원 검색


  
