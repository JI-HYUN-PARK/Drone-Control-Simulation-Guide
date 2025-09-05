# Drone-Control-Simulation-Guide
[Ubuntu 22.04.LTS] Gazebo(garden)_PX4_Ros2(Humble)_QGroundControl_PlotJuggler를 활용한 드론 제어 시뮬레이션 진행

---

## Ubuntu Version Check
[우분투 버전 확인]

    lsb_release -a;
	
* lab_releas: Linux Standard Base에 대한 정보를 보여주는 명령어
<br>

(운영자 버전)

No LSB modules are available.  //LSB 모듈이 설치되어 있지 않음

**Distributor ID**: Ubuntu  // 운영체제의 배포자 ID

**Description**: Ubuntu 22.04.5 LTS  // 현재 시스템에 대한 상세 설명 
LTS: Long Term Support

**Release**: 22.04  // 우분투의 버전 확인 번호

**Codename**: jammy  // 버전과 함께 부여된 코드명

---

## 시뮬레이션 설치 및 연동 가이드 (Gazebo(garden)-PX4-Ros2(Humble)-QGroundControl-PlotJuggler)
### 1. ROS 2 설치
PX4 공식 문서에서는 우분투 22.04에 ROS 2 Humble Hawksbill을 설치하는 것을 권장한다.

#### 1) 로케일 설정
   
    ```
    sudo apt update && sudo apt install locales  
    sudo locale-gen en_US en_US.UTF-8
    sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
    export LANG=en_US.UTF-8
    ```
     
#### 2) ROS 2 저장소 추가
   
   ```
    sudo apt install software-properties-common
    sudo add-apt-repository universe
    sudo apt update && sudo apt install curl -y
    sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
   ```
   
#### 3) ROS 2 Desktop 설치

  ```
    sudo apt update && sudo apt upgrade -y
    sudo apt install ros-humble-desktop ros-dev-tools
  ```

#### 4) 환경설정
터미널을 열 때마다 자동으로 ROS 2 환경을 활성화하기 위해 ~/.bashrc 파일에 다음 명령어를 추가한다.

  ```
    echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
    source ~/.bashrc
  ```


### 2. PX4 SITL & Gazebo 설치
PX4 Autopilot 소스 코드를 다운로드하고 시뮬레이션 환경을 설정한다.

#### 1) PX4-Autopilot 저장소 클론
    
    git clone https://github.com/PX4/PX4-Autopilot.git --recursive
    

#### 2) 설치 스크립트 실행
이 스크립트는 Gazebo를 포함한 필요한 모든 도구들을 설치해준다.
   
    
    bash ./PX4-Autopilot/Tools/setup/ubuntu.sh
    sudo reboot // 생략 가능
    
이 과정에서 Gazebo Harmonic (Gazebo Sim)이 설치된다. 만약 Gazebo Classic이 필요한 경우, 별도로 설치해야 한다.


#### 3) 시뮬레이션 실행
재부팅 후(생략 가능), PX4-Autopilot 디렉토리로 이동하여 Gazebo 시뮬레이션을 실행한다.
    
    cd ~/PX4-Autopilot
    make px4_sitl gz_x500  // 실행
    
이 명령어는 PX4 SITL(Software-In-The-Loop)과 Gazebo 시뮬레이터를 동시에 실행시킨다.
<br>

[참고]

PX4 공식 가이드 <https://docs.px4.io/main/en/sim_gazebo_gz/>를 참고하여 샘플로 들어있는 모델(vehicle)과 환경(World)으로 시뮬레이션 실행이 가능하다.


예) 바람(windy) 시뮬레이션 환경에서 서울의 특정 위치(위도, 경도, 고도)를 홈 위치로 설정하여 PX4 SITL 드론 시뮬레이션을 실행한다.
```
PX4_GZ_WORLD=windy PX4_HOME_LAT=37.418613 PX4_HOME_LON=126.714935 PX4_HOME_ALT=30 make px4_sitl gz_x500
```


### 3. QGroundControl (QGC) 설치
QGC는 AppImage 파일 형태로 배포되어 설치가 간편하다.

#### 1) ModemManager 제거 및 권한 설정
QGC가 시리얼 포트를 사용하기 위해 필요한 설정이다.

	sudo usermod -a -G dialout $USER
	sudo apt-get remove modemmanager -y
	sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y

변경사항을 적용하기 위해 로그아웃 후 다시 로그인하거나 재부팅해야 한다.


#### 2) QGC AppImage 다운로드
PX4 공식 홈페이지<https://docs.qgroundcontrol.com/master/en/qgc-user-guide/getting_started/download_and_install.html>에서 QGroundControl-x86_64.AppImage 파일을 다운로드한다.
이 버전에는 개발용 도구가 포함되어 있어 시뮬레이션에 더 적합하다.

*본인 OS 환경에 맞춰 다운받으면 된다.

(운영자기준 - Ubuntu Linux)
QGroundControl-x86_64.AppImage 파일 다운받았다.


#### 3) 실행 권한 부여 및 실행
다운로드한 파일이 있는 디렉토리로 이동하여 실행 권한을 부여하고 실행한다.

	cd ~/Downloads  // 운영자는 Download 파일에 다운 받음
	chmod +x ./QGroundControl.AppImage  // 실행 권한 부여
	./QGroundControl.AppImage  // 실행
 
QGC가 실행되면 자동으로 Gazebo 시뮬레이터에 연결된 드론을 인식할 것입니다.


### 4. ROS 2와 PX4 연동
PX4는 Micro-XRCE-DDS Agent를 통해 ROS 2와 통신한다.

#### 1) ROS 2 워크스페이스 생성

	mkdir -p ~/ros2_ws/src
	cd ~/ros2_ws/src


#### 2) 필요한 패키지 클론

	git clone https://github.com/PX4/px4_msgs.git
	git clone https://github.com/PX4/px4_ros_com.git


#### 3) Micro-XRCE-DDS-Agent 설치

	git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
	cd Micro-XRCE-DDS-Agent
	mkdir build
	cd build
	cmake ..
	make
	sudo make install
	sudo ldconfig /usr/local/lib/


#### 4) ROS 2 워크스페이스 빌드

	cd ~/ros2_ws
	source /opt/ros/humble/setup.bash
	colcon build


#### 5) 연동 실행
















 

