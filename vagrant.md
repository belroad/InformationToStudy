### vagrant box search
https://atlas.hashicorp.com/boxes/search

### create windows10 box image
https://kyungw00k.github.io/2015/09/09/windows-10-vagrant-box-만들기/

<h1>Mac에 Fullstack 설치하기</h1>
<ol>
  <li>풀스택 디렉토리를 생성한다
    <ol>
      <li>mkdir fullstack</li>
      <li>cd fullstack</li>
    </ol>  
  </li>    
  <li>OPENEDX_RELEASE 환경변수 설정
    <ol>
      <li>export OPENEDX_RELEASE="open-release/eucalyptus.1"</li>
    </ol>
  </li>
  <li>설치 스크립트 다운로드 하기
    <ol>
      <li>curl -OL https://raw.github.com/edx/configuration/$OPENEDX_RELEASE/util/install/install_stack.sh</li>
    </ol>
  </li>
  <li>다운로드 받은 스크립트를 실행하자
    <ol>
      <li>bash install_stack.sh fullstack</li>
    </ol>
  </li>
  <li>vagrant 실행하고 접속하기
    <ol>
      <li>vagrant up</li>
      <li>vagrant ssh</li>
    </ol>
  </li>  
  
  <li>접속후 브라우저에서 http://192.168.33.10/ 로 이동해보면 server error 발생<br/>
        이유는 Ned Batchelder가 풀스택 박스를 잘못 만들었기 때문이다.(df -h 로 empty 확인가능) <br/>
        또는 브라우저 때문일 수도 있다.(크롬을 사용하자) 
    <ol>
      <li>root디렉토리의 empty파일을 지우자. sudo rm -rf /empty</li>      
    </ol>
  </li>    
</ol>

<h1>Windows에 Fullstack 설치하기</h1>
<ol>
  <li>Microsoft Visual C++ 2010 재배포 가능 패키지(x64 및 x86)  설치
    <ul>
      <li>https://www.microsoft.com/ko-kr/download/details.aspx?id=14632</li>
      <li>https://www.microsoft.com/ko-kr/download/confirmation.aspx?id=5555</li>
    </ul>
  </li>
  <li> 관리자 모드로 cmd창 실행
    <ul>
      <li>검색창 -> cmd 입력 -> 명령 프롬프트 실행 아이콘에 우클릭 후 관리자 권한으로 실행</li>
    </ul>
  </li>
  <li>OPENEDX_RELEASE 환경변수 설정
    <ul>
      <li>export OPENEDX_RELEASE="open-release/eucalyptus.1"</li>
    </ul>
  </li>
  <li>설치 스크립트 다운로드 및 설치
    <ul>
      <li>cd \ </li>
      <li>mkdir fullstack</li>
      <li>cd fullstack</li>
      <li>C:\HashiCorp\Vagrant\embedded\bin\curl -L -k https://raw.githubusercontent.com/edx/configuration/master/vagrant/release/fullstack/Vagrantfile > Vagrantfile</li>
      <li>vagrant plugin install vagrant-hostsupdater</li>
      <li>vagrant up</li>
    </ul>
  </li>
</ol>

<h1>precise64 설치</h1>
<ol>
  <li>Ubuntu12 박스추가
    <ul>
      <li>vagrant box add ubuntu/precise64</li>
    </ul>
  </li>
  <li>box 
    <ul>
      <li>vagrant init ubuntu/precise64</li>
    </ul>
  </li>    
  <li> vagrant 설정 
    <ul>
      <li>Vagrantfile 를 vi에디터로 열어서 다음의 주석을 해제 및 수정한다.</li>
      <li>config.vm.network</li>
      <li>config.vm.provider “virtual-box” ~ end</li>
      <li>vb.memory = “2048”</li>
      <li>vb.cpus = “2”</li>
    </ul>
  </li>
  <li>vagrant 실행하고 접속하기
    <ol>
      <li>vagrant up</li>
      <li>vagrant ssh</li>
    </ol>
  </li>  
   <li>precise64 box upgrade 및 update 하기
    <ol>
      <li>upgrade는 맨처음 한번만 해주면 된다. <br/> sudo apt-get upgrade -y</li>
      <li>update는 자주 해줄수록 좋다. <br/> sudo apt-get update -y</li>
    </ol>
  </li>  
  </ol>

#Devstack 설치

1. nfsd 가 정상적으로 작동중인지부터 먼저 확인한다.
   
   sudo nfs status
   
   1) service는 사용가능하나 running이 아닐경우 nfsd를 동작시켜준다.
      
	sudo nfs start

2. Devstack 작업할 디렉토리 생성하기

   Mkdir devstack
 
3. 환경변수를 설정

   export OPENEDX_RELEASE="open-release/eucalyptus.2”

4. 설치 스크립트 다운로드	
	
   curl -OL https://raw.github.com/edx/configuration/$OPENEDX_RELEASE/util/install/install_stack.sh

5. devstack 가성머신을 생성하고 시작하기 위해 설치 스크립트 실행

   bash install_stack.sh devstack
	
6. devstack 가성머신에 접속하기

   vagrant ssh

7. 에덱스어플 유저로 접속하기 / 관리자계정으로 모든 어플 시작

   sudo su edxapp      / supervisorctrl start all	 

8. 에덱스 LMS 시작하기

   paver devstack lms

