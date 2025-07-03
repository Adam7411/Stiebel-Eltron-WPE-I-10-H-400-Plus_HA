Servicewelt Stiebel Eltron + Home Assistant


![ChatGPT Image May 18, 2025, 07_18_35 PM](https://github.com/user-attachments/assets/c74c7744-5b17-4d73-bed7-3b574b2c934c)

SOFTWARE POMPY 15.1.17

![Bez tytułu](https://github.com/user-attachments/assets/52de6584-1345-4f77-8788-380b35f6a730)


Testowane na panelu lokalnym sterowania Servicewelt pompy Stiebel Eltron WPE-I 10 H 400 Plus, moduł ISG niełączy się z tą pompą przez CAN. 

![WPE-I 10 H 400 PLUS](https://github.com/user-attachments/assets/a3c94749-cb8b-4e89-ba24-e2fa1e06cecf)



Servicewelt ma być bez hasła oraz nazwy użytkownika bramy ISG.
Po lokalnym adresie modułu ISG w sieci domowej z strony panelu sterowania wyciągnięde narzędziem developerskim kilka głównych danych do Home Assistant (mało ale moj panel ISG dużo niedaje).

Główny zamiar odczyt podstawowych temparatur, sterowanie pompą i z czasem automatyzacje w HA itp.


Wszystkie adresy URL w pliku configuration.yaml korzystają z adresu http://192.168.100.126. Jeśli twój ISG ma inny adres, podmień go wszędzie na swój adres. (domyslny adres zazwyczaj jest 192.168.1.126)



![stiebel1](https://github.com/user-attachments/assets/f7b1c5e7-b89a-49ca-8c60-8cd553de6ad0)



![screenshoteasy (2)](https://github.com/user-attachments/assets/e4b19753-3fd1-4ede-9add-28b496affe6e)

Do pliku configuration.yaml , kopiujemy kod podmieniamy na swój lokalny adres modułu ISG
```yaml
rest_command:

  set_stiebel_value:
    url: "http://192.168.100.134/save.php"
    method: POST
    headers:
      Content-Type: application/x-www-form-urlencoded
    payload: 'data=[{"name":"{{ name }}","value":"{{ value }}"}]'

  # BEZ ZMIAN: Te komendy zostają, bo pobierają wartości z input_number
  ustaw_temp_cwu:
    url: "http://192.168.100.134/save.php"
    method: POST
    headers:
      Content-Type: application/x-www-form-urlencoded
    payload: >
      data=[{"name":"val40023","value":"{{ states('input_number.temp_cwu_on') | replace('.', ',') }}"},
            {"name":"val40024","value":"{{ states('input_number.temp_cwu_off') | replace('.', ',') }}"}]

  zapisz_temp_pomieszczenia:
    url: "http://192.168.100.134/save.php"
    method: POST
    headers:
      Content-Type: application/x-www-form-urlencoded
    payload: >
      data=[{"name":"val40006","value":"{{ states('input_number.temp_pomieszczenia') | round(2) | string | replace('.', ',') }}"}]


switch:
  - platform: template
    switches:
      tryb_cwu:
        friendly_name: "Tryb CWU"
        # Stan przełącznika jest odczytywany z Twojego sensora binarnego
        value_template: "{{ is_state('binary_sensor.stiebel_eltron_stan_cwu', 'on') }}"
        # Akcja po włączeniu przełącznika
        turn_on:
          service: rest_command.set_stiebel_value
          data:
            name: "val9"
            value: "1"
        # Akcja po wyłączeniu przełącznika
        turn_off:
          service: rest_command.set_stiebel_value
          data:
            name: "val9"
            value: "0"
        icon_template: >-
          {% if is_state('binary_sensor.stiebel_eltron_stan_cwu', 'on') %}
            mdi:water-pump
          {% else %}
            mdi:water-pump-off
          {% endif %}

      tryb_ogrzewania:
        friendly_name: "Tryb ogrzewania"
        value_template: "{{ is_state('binary_sensor.pompa_stiebel_eltron_stan_ogrzewania', 'on') }}"
        turn_on:
          service: rest_command.set_stiebel_value
          data:
            name: "val10"
            value: "1"
        turn_off:
          service: rest_command.set_stiebel_value
          data:
            name: "val10"
            value: "0"
        icon_template: >-
          {% if is_state('binary_sensor.pompa_stiebel_eltron_stan_ogrzewania', 'on') %}
            mdi:radiator
          {% else %}
            mdi:radiator-off
          {% endif %}

      antylegionella:
        friendly_name: "Antylegionella"
        value_template: "{{ is_state('binary_sensor.stiebel_eltron_antylegionella_stan', 'on') }}"
        turn_on:
          service: rest_command.set_stiebel_value
          data:
            name: "val25"
            value: "1"
        turn_off:
          service: rest_command.set_stiebel_value
          data:
            name: "val25"
            value: "0"
        icon_template: >-
          {% if is_state('binary_sensor.stiebel_eltron_antylegionella_stan', 'on') %}
            mdi:shield-check
          {% else %}
            mdi:shield-outline
          {% endif %}

input_number:
  temp_cwu_on:
    name: Temperatura załączenia CWU
    min: 30
    max: 60
    step: 0.5
    unit_of_measurement: "°C"
    mode: box
    initial: 43

  temp_cwu_off:
    name: Temperatura wyłączenia CWU
    min: 30
    max: 70
    step: 0.5
    unit_of_measurement: "°C"
    mode: box
    initial: 48

  temp_pomieszczenia:
    name: Temperatura pomieszczenia
    min: 10
    max: 30
    step: 0.1
    unit_of_measurement: "°C"
    mode: box
    initial: 18.5
    
binary_sensor:
  - platform: rest
    name: "Stiebel Eltron Stan CWU"
    resource: http://192.168.100.134/?s=3,3
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {% set val = value.split('id="aval9" value="')[1].split('"')[0] %}
      {{ val == "ZAL" }}
    device_class: power
    scan_interval: 5


  - platform: rest
    name: "Pompa Stiebel Eltron stan ogrzewania"
    resource: http://192.168.100.134/?s=0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {% set lines = value.split('\n') %}
      {% for line in lines %}
        {% if 'id="aval10"' in line %}
          {% set val = line.split('value="')[1].split('"')[0] %}
          {{ val == "OGRZEWANIE" }}
        {% endif %}
      {% endfor %}
    device_class: power
    scan_interval: 10


  - platform: rest
    name: "Stiebel Eltron Antylegionella stan"
    resource: http://192.168.100.134/?s=3,3
    method: GET
    value_template: >
      {% if 'id="aval25"' in value %}
        {% set val = value.split('id="aval25" value="')[1].split('"')[0] %}
        {{ val == "ZAL" }}
      {% else %}
        false
      {% endif %}
    device_class: power
    scan_interval: 10

sensor:
# Stiebel Eltron

  - platform: rest
    name: "Temperatura zasobnika na dole Stiebel Eltron"
    resource: http://192.168.100.134/?s=2,7
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temperatura zasobnika na dole</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('°C', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "°C"

  - platform: rest
    name: "Temperatura rury wylotowej Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temp. rury wylotowej</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('°C', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "°C"

  - platform: rest
    name: "Temperatura wlotu skraplacza Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temp. wlotu skraplacza</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('°C', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "°C"

  - platform: rest
    name: "Temperatura wylotu skraplacza Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temp. wylotu skraplacza</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('°C', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "°C"

  - platform: rest
    name: "Temperatura wlotu solanki Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temp. wlotu solanki</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('°C', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "°C"

  - platform: rest
    name: "Temperatura wylotu solanki Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temp. wylotu solanki</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('°C', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "°C"

  - platform: rest
    name: "Temperatura zewnętrzna Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temperatura zewnętrzna</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('°C', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "°C"

  - platform: rest
    name: "Temperatura pomieszczenia Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temp. pomieszczenia</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('°C', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "°C"

  - platform: rest
    name: "Temperatura skraplacza (manometryczna) Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temp. skraplacza (manometryczna)</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('°C', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "°C"

  - platform: rest
    name: "Temperatura przegrzania Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temp. przegrzania</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('K', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "K"

  - platform: rest
    name: "Temperatura wychłodzenia Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Temp. wychłodzenia</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('K', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "K"


  - platform: rest
    name: "Prędkość obr. sprężarki Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Procentowa prędkość obr. sprężarki</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('%', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "%"

  - platform: rest
    name: "Stopnie sprężarki Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Dostępnie stopnie sprężarki</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('gear', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "gear"

  - platform: rest
    name: "Zawór mieszający ZOD Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Sygnał ster. zaworu mieszającego ZOD</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('%', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "%"

  - platform: rest
    name: "Prędkość pompy solanki Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Prędkość pompy obiegowej solanki</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('%', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "%"

  - platform: rest
    name: "Ciśnienie niskie Stiebel Eltron"
    resource: http://192.168.100.134/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Strona niskiego ciśnienia, ciśnienie</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('bar (g)', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "bar"

```



do scripts.yaml
```yaml
zapisz_temperatury_cwu:
  alias: Zapisz temperatury CWU
  sequence:
  - service: rest_command.ustaw_temp_cwu
  mode: single
zapisz_temp_pomieszczenia:
  alias: Zapisz temperaturę pomieszczenia
  sequence:
  - service: rest_command.zapisz_temp_pomieszczenia
  mode: single

```
Karty sterowania w HA itp
```yaml
type: vertical-stack
cards:
  - type: entities
    title: Sterowanie Pompą Ciepła
    show_header_toggle: false
    entities:
      - entity: sensor.temperatura_zasobnika_na_dole_stiebel_eltron
        name: Temp. zbiornika CWU
      - entity: switch.tryb_cwu
        name: Ciepła Woda Użytkowa
      - type: divider
      - entity: switch.tryb_ogrzewania
        name: Ogrzewanie Domu
      - type: divider
      - entity: switch.antylegionella
        name: Program Antylegionella
    state_color: true
  - type: entities
    title: Ustawienia Temperatur
    show_header_toggle: false
    entities:
      - entity: input_number.temp_cwu_on
        name: Temp. załęczenia CWU
      - entity: input_number.temp_cwu_off
        name: Temp. wyłączenia CWU
      - type: call-service
        name: Zapisz temperatury CWU
        icon: mdi:thermometer-check
        action_name: Zapisz
        service: script.zapisz_temperatury_cwu
      - type: divider
      - entity: input_number.temp_pomieszczenia
        name: Temp. pomieszczenia
      - type: call-service
        name: Zapisz temperaturę pomieszczenia
        icon: mdi:home-thermometer-outline
        action_name: Zapisz
        service: script.zapisz_temp_pomieszczenia


```



```yaml
type: entities
entities:
  - automation.zmien_stan_cwu_stiebel
  - sensor.cisnienie_niskie_stiebel_eltron
  - sensor.predkosc_obr_sprezarki_stiebel_eltron
  - sensor.predkosc_pompy_solanki_stiebel_eltron
  - binary_sensor.stiebel_eltron_stan_cwu
  - sensor.stopnie_sprezarki_stiebel_eltron
  - sensor.temperatura_pomieszczenia_stiebel_eltron
  - sensor.temperatura_przegrzania_stiebel_eltron
  - sensor.temperatura_rury_wylotowej_stiebel_eltron
  - sensor.temperatura_skraplacza_manometryczna_stiebel_eltron
  - sensor.temperatura_wlotu_skraplacza_stiebel_eltron
  - sensor.temperatura_wlotu_solanki_stiebel_eltron
  - sensor.temperatura_wychlodzenia_stiebel_eltron
  - sensor.temperatura_wylotu_skraplacza_stiebel_eltron
  - sensor.temperatura_wylotu_solanki_stiebel_eltron
  - sensor.temperatura_zasobnika_na_dole_stiebel_eltron
  - sensor.temperatura_zewnetrzna_stiebel_eltron
  - sensor.zawor_mieszajacy_zod_stiebel_eltron
```




