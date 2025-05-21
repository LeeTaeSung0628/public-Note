# DispatcherServlet의 역할
	- 웹 애플리케이션 최전방에서 사용자의 요청을 접수하여 URL기주능로 요청을 처리할 controller를 찾고, 그 controller에 처리를 위임한 후, 결과를 받아서 사용자에게 처리 결과가 담긴 화면을 제공해준다.

- 설정은 web.xml의 정보를 활용한다. 사용자 요청을 처리할 Controller목록과 사용자 에게 보여줄 화면을 찾는 ViewResolver가 있다.

---