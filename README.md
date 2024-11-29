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
  ### code
    <details>
        <summary>chatController.java</summary>
        
        @Controller
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
        	    
        	    return chatService.getMessagesByChatRoomId(chatRoomId)
        	            .thenApply(messages -> {
        	                model.addAttribute("messages", messages);
        	                return "member/chatRoom";
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

            // 채팅방 삭제
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

    <details>
        <summary>chatRoom.jsp</summary>

        import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.4/firebase-app.js'
        import { getDatabase, ref, onValue, set, child, push, onChildAdded, query, limitToLast } 
        	from 'https://www.gstatic.com/firebasejs/10.12.4/firebase-database.js'
    
        // Firebase 초기화
    	const firebaseConfig = {
    		// firebase 서비스키 입력
    	};
    
        const app = initializeApp(firebaseConfig);
    	var db = getDatabase(app);
    
    	// Connect function 수정
    	function connect() {
       		const chatRoomId = document.getElementById("chatRoomId").value; // 서버에서 전달된 chatRoomId 사용
        	const dbRef = ref(db, 'messages/chatRooms/' + chatRoomId);
    
        	// 새로운 데이터 추가 시 실시간으로 화면에 표시
        	onChildAdded(dbRef, (data) => {
            	var name = data.val().sender;
    			var msg = data.val().message;
    			console.log(name);
    			console.log(msg);
            	appendMessage(name, msg);
        	});
    	}
       
    	function appendMessage(name,msg) {
    		const Id = document.getElementById("Id").value;
    
    		if (name == Id) {
            	$('#chatMessageArea').append("<div class='myId'> " + name + "</div>" +
                                         "<div class='myMsg'><span class='Msg'>" + msg + "</span></div>");
       		 } else {
            	$('#chatMessageArea').append("<div class='yourId'> " + name + "</div>" +
                                         "<div class='yourMsg'><span class='sendMsg'>" + msg + "</span></div>");
        	}
        	
        	const chatAreaHeight = $('#chatArea').height();
        	const maxScroll = $('#chatMessageArea').height() - chatAreaHeight;
        	$('#chatArea').scrollTop(maxScroll);
    	}
    
    	$(document).ready(function() {
            connect();
       	});
        </script>
    </details>


    
- Oracle 기반 계층형 구조와 MyBatis를 활용한 문의 게시판 CRUD 구현
    - Spring Security을 이용해 일반 사용자는 문의글을 작성하고, 관리자는 답변을 작성할 수 있도록 권한을 설정하였습니다.
    - 문의 게시판의 CRUD 기능과 비밀글 기능을 구현하여 사용자별 접근 권한을 강화했습니다.
    - 비밀번호 변경은 AJAX를 이용해 비동기 방식으로 서버에 요청하며 변경 완료시 입력란과 버튼을 비활성화 하여 변경 완료 상태를 시각적으로 전달하였습니다.

  ### code
    <details>
        <summary>inquiryBoardController.java</summary>

        @Controller
        public class inquiryBoardController
        {
        	@Autowired
        	inquiryBoardSevice bbs;
        	@Autowired
        	ServletContext context;
        	
        
        	// 문의 게시판 목록
        	@RequestMapping("guest/inquiryBoard") 
        	public String inquiryBoard(HttpServletRequest request, Model model)
        	{
        		Map<String, Object> map = new HashMap<String, Object>();
                
                // 검색
        		String searchField = request.getParameter("searchField");
        		String searchWord = request.getParameter("searchWord");
        		
                if (searchWord != null) {
                    map.put("searchField", searchField);
                    map.put("searchWord", searchWord);
                }
                
              // 페이징
              int totalCount = bbs.listCountDao(map); // 게시글 총 갯수
              int pageSize = 10; // 한 페이지 불러올 페이지
              int blockPage = 5; // 블럭 갯수 
              int pageNum = 1; // 목록 첫 진입시 무조건 1 페이지 
              
              String pageTemp = request.getParameter("pageNum"); 
              if (pageTemp != null && !pageTemp.equals(""))
              	pageNum = Integer.parseInt(pageTemp);
              
              int start = (pageNum - 1) * pageSize + 1;	// 첫 게시물 번호
              int end = pageNum * pageSize;	// 마지막 게시물 번호
              
              map.put("start", start);
              map.put("end", end);
              
              String pagingImg = BoardPage.pagingStr(totalCount, pageSize,
                      blockPage, pageNum, "../guest/inquiryBoard", searchField, searchWord);
        
              String sId = SecurityContextHolder.getContext().getAuthentication().getName();
              
              
              model.addAttribute("Id", sId); // 로그인된 아이디
        	  model.addAttribute("searchField", searchField); // 받아온 검색필드
        	  model.addAttribute("searchWord", searchWord); // 받아온 검색어
              model.addAttribute("pagingImg", pagingImg); // 목록 하단에 출력할 페이지 번호
              model.addAttribute("totalCount", totalCount); // 전체 게시물 갯수
              model.addAttribute("pageSize", pageSize); // 한 페이지당 출력할 게시물 갯수(설정값)
              model.addAttribute("pageNum", pageNum); // 현재 페이지 번호 
              model.addAttribute("list", bbs.listDao(map)); 
              
              return "guest/inquiryBoard";
        	}
        	
        	// 게시글 비밀번호 창 
        	@RequestMapping("member/inquiryBoardPass")
        	public String inquiryBoardPasss(HttpServletRequest request, Model model)
        	{
        		String sId = SecurityContextHolder.getContext().getAuthentication().getName();
        		String idx = request.getParameter("idx");
        		inquiryBoardDto dto  =  bbs.viewDao(idx);
        
        		model.addAttribute("dto", dto);
        		model.addAttribute("Id", sId); // 로그인된 아이디
        		model.addAttribute("idx", idx);
        		
        		return "member/inquiryBoardPass";
        	}
        	
        	//문의 게시판 상세보기
        	@RequestMapping("member/inquiryBoardview")
        	public String inquiryBoardview(HttpServletRequest request,Model model)
        	{
        		String sId = SecurityContextHolder.getContext().getAuthentication().getName();
        		String password = request.getParameter("memberBoardPassword");
        		
        		//상세 보기
        		String idx = request.getParameter("idx");
        		inquiryBoardDto dto  =  bbs.viewDao(idx);
        		model.addAttribute("dto", dto);
        		
        		// 조회수
        		bbs.viewCountDao(idx);  
        		
        		// 파일 불러오기
        		String ext = null, fileName = dto.getSfile();
        		if (fileName != null) {
        			ext = fileName.substring(fileName.lastIndexOf(".")+1);
        		}
        		String[] mimeStr = {"png", "jpg", "gif", "PNG","jpeg", "bmp"};
        		List<String> mimeList = Arrays.asList(mimeStr);
        		boolean isImage=false;
        		if(mimeList.contains(ext)) {
        			isImage=true;
        		}
        		
        		model.addAttribute("isImage", isImage);
        		model.addAttribute("Id", sId);
        		model.addAttribute("memberPassword", password);
        		
        		return "member/inquiryBoardview";
        	}
        	
        	//문의 게시판 글쓰기폼
        	@RequestMapping("member/inquiryBoardWriteForm")
        	public String inquiryBoardWriteForm(Model model)
        	{
        		String userId =SecurityContextHolder.getContext().getAuthentication().getName();		
        		model.addAttribute("userId", userId);
        		
        		return "member/inquiryBoardWriteForm";
        	}
        	
        	//문의 게시판 글쓰기
        	@RequestMapping("member/inquiryBoardWrite")
        	public String inquiryBoardWrite(HttpServletRequest request, @RequestParam("ofile") MultipartFile file) throws FileNotFoundException
        	{	
        		// 파일 업로드
        		String ofileName = file.getOriginalFilename();
        		String uploadDir = context.getRealPath("/static/files");
        		String sfileName = "";
        		
        		File dir = new File(uploadDir);
        		if (!dir.exists()) {
        	        dir.mkdirs();
        	    }
        		sfileName = UUID.randomUUID().toString() + "_" + ofileName;
        		
        		
        		File destination = new File(dir,sfileName);
        		try {
        			file.transferTo(destination);
        			
        		} catch (IOException e) {
        			e.printStackTrace();
        			return "redirect:/member/inquiryBoardWriteForm?status=fail";
        		}
        		
        		String sId = SecurityContextHolder.getContext().getAuthentication().getName();
        		request.getParameter("title");
        		request.getParameter("content");
        		request.getParameter("boardPass");	
        		
        		 bbs.writeDao(sId,
        					   request.getParameter("title"),
        					   request.getParameter("content"),
        					   ofileName,
        					   sfileName,
        					   request.getParameter("boardPass"));
        		
        		return "redirect:../guest/inquiryBoard";
        	}	
        	
        	//문의 게시판 답글 글쓰기 폼
        	@RequestMapping("admin/inquiryBoardReplyWriteForm")
        	public String inquiryBoardReplyWriteForm(HttpServletRequest request,Model model)
        	{
        		String xIdx = request.getParameter("idx");
        		
        		model.addAttribute("dto", bbs.viewDao(xIdx));
        		return "admin/inquiryBoardReplyWriteForm";
        	}
        	
        	//문의 게시판 답변 글쓰기
        	@RequestMapping("admin/inquiryBoardReplyWrite")
        	public String replyWriteDao(HttpServletRequest request, @RequestParam("ofile") MultipartFile file ) throws Exception
        	{		
        		Map<String, String> map = new HashMap<String, String>();	 
        		
        		// 파일 업로드
        		String ofileName = file.getOriginalFilename();
        		String uploadDir = context.getRealPath("/static/files");
        		
        		File dir = new File(uploadDir);
        		if (!dir.exists()) {
        	        dir.mkdirs();
        	    }
        		String sfileName = UUID.randomUUID().toString() + "_" + ofileName;
        		
        		File destination = new File(dir,sfileName);
        		try {
        			file.transferTo(destination);
        			
        		} catch (IOException e) {
        			e.printStackTrace();
        			return "redirect:/member/inquiryBoardReplyWrite?status=fail";
        		}
        		
        		String sId = request.getParameter("id");
        		String smemberId = request.getParameter("memberid");
        		String sTitle = request.getParameter("title");
        		String sContent = request.getParameter("content");
        		String sBoardPass = request.getParameter("boardPass");		
        		String sIdx = request.getParameter("idx");
        		map.put("item1", sId);
        		map.put("item2", smemberId);
        		map.put("item3", sTitle);
        		map.put("item4", sContent);
        		map.put("item5", ofileName);
        		map.put("item6", sfileName);
        		map.put("item7", sBoardPass);
        		map.put("item8", sIdx);
        		
        		int nResult = bbs.replyWriteDao(map);
        		//System.out.println("Write : " + nResult);
        		bbs.responsesCountDao(sIdx);
        		
        		return "redirect:../guest/inquiryBoard";
        	}
        	
        	//문의 게시글 삭제
        	@RequestMapping("member/inquiryBoardDelete")
        	public String inquiryBoardDelete(HttpServletRequest request, Model model)
        	{
        		String idx = request.getParameter("idx");
        
        		model.addAttribute("dto", bbs.deleteDao(idx));
        		return "redirect:../guest/inquiryBoard";
        	}
        	
        	@RequestMapping("member/inquiryBoardEditorForm")
        	public String inquiryBoardEditorForm(HttpServletRequest request, Model model)
        	{
        		String idx = request.getParameter("idx");
        		inquiryBoardDto dto = bbs.viewDao(idx);
        		
        		model.addAttribute("dto", dto);
        		model.addAttribute("existingOfile", dto.getOfile());
        		model.addAttribute("existingSfile", dto.getSfile());
        		
        		return "member/inquiryBoardEditorForm";
        	}
        	
        	//문의 게시판 수정
        	@RequestMapping("member/inquiryBoardEditor")
        	public String inquiryBoardEditor(HttpServletRequest request,@RequestParam("ofile") MultipartFile file)
        	{
        		String idx = request.getParameter("idx");		
        		
        		inquiryBoardDto dto = bbs.viewDao(idx);
        		String ofileName = dto.getOfile();
        		String sfileName = dto.getSfile();
        		
        		if (!file.isEmpty()) {
        			ofileName = file.getOriginalFilename();
        			sfileName = UUID.randomUUID().toString() + "_" + ofileName;
        			String uploadDir = context.getRealPath("/static/files");
        
        			File dir = new File(uploadDir);
        			if (!dir.exists()) {
        		        dir.mkdirs();
        		    }
        			
        			File destination = new File(dir,sfileName);
        			try {
        				file.transferTo(destination);
        			} catch (IllegalStateException e) {
        				e.printStackTrace();
        			} catch (IOException e) {
        				e.printStackTrace();
        			}
        		}
        		
        		String Id = dto.getId();
        		String Title = request.getParameter("title");
        		String Content = request.getParameter("content");
        		
        		Map<String, String> map = new HashMap<String, String>();
        		map.put("mId", Id);
        		map.put("mTitle", Title);
        		map.put("mContent", Content);
        		map.put("moFileName", ofileName);
        		map.put("msFileName", sfileName);
        		map.put("mIdx", idx);
        		
        		bbs.editorDao(map);
        		return "redirect:../guest/inquiryBoard";
        	}
        	
        	@RequestMapping("member/changeBoardPass")
        	public @ResponseBody String changeBoardPass(HttpServletRequest request, @RequestParam("idx") String idx, @RequestParam("boardPass") String boardPass)
        	{
        		bbs.changeBoardPass(idx, boardPass);
        
        		return "비밀번호가 성공적으로 변경되었습니다.";
        	}        	
        }
    </details>

    <details>
        <summary>IinquiryBoardDao.java</summary>

        @Mapper
        public interface IinquiryBoardDao {
        	
        	public List<inquiryBoardDto> listDao(Map<String, Object> map);
        	public int listCountDao(Map<String, Object> map);
        	public inquiryBoardDto viewDao(@Param("_idx") String idx);
        	public int writeDao(@Param("id")String id,@Param("title") String title,@Param("content") String content, 
        			@Param("ofile") String ofile, @Param("sfile")String sfile, @Param("boardPass") String boardPass);
        	public int replyWriteDao(Map<String, String> map);
        	public int deleteDao(String id);
        	public int viewCountDao(@Param("_idx")String idx);
        	public int responsesCountDao(@Param("_idx")String idx);
        	public int editorDao(Map<String, String> map);
        	public int changeBoardPass(@Param("_idx")String idx, @Param("_boardPass")String boardPass);
        	
        }
    </details>    

    <details>
        <summary>inquiryBoardDao.xml</summary>

        <mapper namespace="com.project.springboot.dao.IinquiryBoardDao">
		<select id="listDao" resultType="com.project.springboot.dto.inquiryBoardDto">
			SELECT * FROM (
            SELECT Tb.*, ROWNUM rNum FROM (
            SELECT * FROM inquiryBoard
	     		<if test="searchWord != null and searchWord != ''">
							WHERE ${searchField} Like '%${searchWord}%'
							OR parentId Like '%${searchWord}%'
				</if>            
            START WITH parentIdx=0
            CONNECT BY PRIOR idx = parentIdx
            ORDER SIBLINGS BY idx DESC
            ) Tb )
            WHERE rNum BETWEEN ${start} AND ${end}
        </select>

		
		<select id="listCountDao" resultType="int">
			select count(*) from inquiryBoard 
				<if test="searchWord != null and searchWord != ''">
					where ${searchField} 
					Like '%${searchWord}%'
				</if>
		</select>
		
		<select id="viewDao" resultType="com.project.springboot.dto.inquiryBoardDto">
			select * from inquiryBoard where idx = #{_idx}
		</select>
		
		<insert id="writeDao" parameterType="java.util.HashMap">
			insert into inquiryBoard (idx, id, title, content, ofile, sfile, isAnnouncement, boardPass)
					values (inquiryBoard_seq.nextval, #{id}, #{title}, #{content}, #{ofile, jdbcType=VARCHAR}, #{sfile, jdbcType=VARCHAR}, 'N', #{boardPass})			
		</insert>
		
		<insert id="replyWriteDao" parameterType="java.util.HashMap">
			insert into inquiryBoard (idx, id, parentId, title, content, ofile, sfile, isAnnouncement, boardPass, parentIdx)
			values (inquiryBoard_seq.nextval, #{item1}, #{item2}, #{item3}, #{item4}, #{item5, jdbcType=VARCHAR}, #{item6, jdbcType=VARCHAR}, 'Y', #{item7}, #{item8})
		</insert>
		
		<delete id="deleteDao">
			delete from inquiryBoard where idx = #{_idx}
		</delete>
		
		<update id="viewCountDao">
			update inquiryBoard SET viewCount=viewCount+1 where idx = #{_idx}
		</update>
		
		<update id="editorDao">
			update inquiryBoard 
				SET id = #{mId},
					title = #{mTitle},
					content = #{mContent},
					ofile = #{moFileName, jdbcType=VARCHAR},
					sfile = #{msFileName, jdbcType=VARCHAR}
				where idx = #{mIdx}
		</update>
		
		<update id="responsesCountDao">
			update inquiryBoard SET responses=responses+1 where idx = #{_idx}
		</update>
		
		<select id="getTotalCount" resultType="int">
	        SELECT COUNT(*) FROM inquiryBoard
	    </select>
	    
	    <update id="changeBoardPass">
	    	update inquiryBoard 
		    SET boardPass= #{_boardPass}
		    where idx = #{_idx} or parentIdx = #{_idx}
	    </update>
		
	</mapper>
    </details>    

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


  
