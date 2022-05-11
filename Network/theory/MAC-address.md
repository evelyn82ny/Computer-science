> 'IT 엔지니어를 위한 네트워크 입문' 내용을 정리한 글 입니다.

# MAC 주소

MAC (Media Access Control) 주소는 data link layer (2계층) 에서 통신을 위해 NIC (Network Interface Controller(card)) 에 할당된 고유 식별자이다.<br>
즉, 네트워크에 접속하는 모든 장비는 물리적인 주소인 MAC 주소로 통신한다.<br>

MAC 주소는 단말에 종속되는 것이 아니라 **NIC 에 종속**된다. 멀티레이어 스위치, 라우터와 같은 복잡한 네트워크 장비는 1개의 장비에 여러개의 NIC 를 갖고 있기 때문에 1개의 장비에 여러개의 MAC 주소가 할당될 수 있다.<br>
<br>

## MAC 주소 체계

MAC 주소는 변경할 수 없도록 하드웨어에 고정되어 출하된다.<br>
48비트의 16진수 12자리로 표현되고, 앞 24비트를 OUI(Organizational Unique Identifier), 뒤 24비트를 UAA(Universally Administered Address) 값이다.<br>

- OUI : IEEE 가 모든 장비에 대한 주소를 확인할 수 없어 제조업체에게 주소 풀을 할당하는데 이를 **제조사 코드 (vendor code)** 라고 한다.
- UAA : 뒤 24비트는 각 제조사에서 자체적으로 할당한다.

네크워크 카드나 장비를 생산할 때 하드웨어 적으로 정해져 나오므로 MAC 주소를 BIA(Burned in Address)라고도 부른다.<br>

### MAC 주소 중복되면 문제되나?

MAC 주소가 유니크하다고 하지만 실수로 중복될 수도 있다. 하지만 동일 네트워크에서만 중복되지 않으면 문제되지 않는다.<br>

- 동일 네트워크 : 동일 네트워크에서 같은 MAC 주소를 갖고 있으면 문제된다.
- 다른 네트워크 : 다른 네트워크로 넘어가기 위해 라우터의 도움을 받을 땐 출발지, 도착지의 MAC 주소가 변경되므로 **다른 네트워크 상에서 같은 MAC 주소를 갖고 있어도 문제되지 않는다.**
<br>

## MAC 주소 동작

1계층으로부터 전기신호가 들어오면 2계층에서 packet (데이터 형태)으로 변환하여 내용을 구분한 후 도착지 MAC 주소를 확인한다. NIC 는 자신의 MAC 주소와 다르면 packet 을 버린다.<br>
하지만 도착지 주소가 자신의 MAC 주소와 동일하거나 브로트캐스트, 멀티캐스트 같은 그롭 주소이면 해당 패킷을 처리해 상위 계층으로 넘겨야한다.<br>
NIC 가 패킷을 처리하는게 아니고 OS나 application 에서 처리하므로 시스템에 부하가 작용한다.<br>

### 무차별 모드 Promiscuous Mode

네트워크 상태를 모니터링하거나 디버그, 분석 용도로 전체 패킷을 수집해 분석해야되는 상황에서 NIC 가 정상적으로 동작하면 패킷을 버리는 경우가 생긴다.<br>
이럴경우 패킷을 분석할 수 없으므로 자신의 MAC 주소와 상관없는 패킷이 들어와도 분석할 수 있도록 NIC 를 무차별 모드로 구성한다.<br>
무차별 모드를 사용하는 대표적인 application 으로 네트워트 패킷 분석 wireshark 이다.<br>