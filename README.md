P.S stworzone amatorsko z pomocą AI 

Testowane na panelu sterowania Stiebel Eltron WPE-I 10 H 400 Plus
po lokalnym adresie modułu ISG w sieci domowej z strony panelu sterowania wyciągnięde narzędziem developerskim kilka głównych danych do Home Assistant (mało ale moj panel ISG dużo niedaje)
Główny zamiar odczyt podstawowych temparatur sterowanie pompą i z czasem automatyzacje w HA.



Wszystkie adresy URL w pliku configuration.yaml korzystają z adresu http://192.168.100.126. Jeśli twój ISG ma inny adres, podmień go wszędzie na swój adres. (zazwyczaj jest 192.168.1.126)



![screenshoteasy](https://github.com/user-attachments/assets/706574bf-ad7a-48af-915c-75855bab6ace)


Do pliku configuration.yaml , podmieniamy na swój lokalny adres modułu ISG
```yaml
rest_command:
  zapisz_cwu:
    url: "http://192.168.100.126/save.php"
    method: POST
    headers:
      Content-Type: application/x-www-form-urlencoded
    payload: >
      data=[{"name":"val9","value":"{{ '1' if is_state('input_select.tryb_cwu', 'ZAL') else '0' }}"},
            {"name":"val15","value":"0"},
            {"name":"val25","value":"1"}]

  ustaw_temp_cwu:
    url: "http://192.168.100.126/save.php"
    method: POST
    headers:
      Content-Type: application/x-www-form-urlencoded
    payload: >
      data=[{"name":"val40023","value":"{{ states('input_number.temp_cwu_on') | replace('.', ',') }}"},
            {"name":"val40024","value":"{{ states('input_number.temp_cwu_off') | replace('.', ',') }}"}]

  zapisz_tryb_ogrzewania:
    url: "http://192.168.100.126/save.php"
    method: POST
    headers:
      Content-Type: application/x-www-form-urlencoded
    payload: >
      data=[{"name":"val10","value":"{{ '1' if is_state('input_select.tryb_ogrzewania', 'OGRZEWANIE') else '0' }}"}]

  zapisz_temp_pomieszczenia:
    url: "http://192.168.100.126/save.php"
    method: POST
    headers:
      Content-Type: application/x-www-form-urlencoded
    payload: >
      data=[{"name":"val40006","value":"{{ states('input_number.temp_pomieszczenia') | round(2) | string | replace('.', ',') }}"}]

  zapisz_antylegionella:
    url: "http://192.168.100.126/save.php"
    method: POST
    headers:
      Content-Type: application/x-www-form-urlencoded
    payload: >
      data=[{"name":"val25","value":"{{ '1' if is_state('input_select.antylegionella', 'ZAL') else '0' }}"}]

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


input_select:
  tryb_cwu:
    name: Tryb CWU
    options:
      - WYL
      - ZAL
    initial: WYL

  tryb_ogrzewania:
    name: Tryb ogrzewania
    options:
      - WYŁ.
      - OGRZEWANIE
    initial: WYŁ.


  antylegionella:
    name: Antylegionella
    options:
      - WYL
      - ZAL
    initial: WYL

# Stiebel Eltron
sensor:
  - platform: rest
    name: "Temperatura zasobnika na dole Stiebel Eltron"
    resource: http://192.168.100.126/?s=2,7
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
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
    resource: http://192.168.100.126/?s=1,0
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {{ value.split('Strona niskiego ciśnienia, ciśnienie</td>')[1]
              .split('<td class="value">')[1]
              .split('</td>')[0]
              .replace('bar (g)', '')
              .replace(',', '.') | float }}
    unit_of_measurement: "bar"


binary_sensor:
  - platform: rest
    name: "Stiebel Eltron Stan CWU"
    resource: http://192.168.100.126/?s=3,3
    method: GET
    # Usunięto nagłówek Cookie
    value_template: >
      {% set val = value.split('id="aval9" value="')[1].split('"')[0] %}
      {{ val == "ZAL" }}
    device_class: power
    scan_interval: 30


  - platform: rest
    name: "Pompa Stiebel Eltron stan ogrzewania"
    resource: http://192.168.100.126/?s=0
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
    scan_interval: 30


  - platform: rest
    name: "Stiebel Eltron Antylegionella stan"
    resource: http://192.168.100.126/?s=3,3
    method: GET
    value_template: >
      {% if 'id="aval25"' in value %}
        {% set val = value.split('id="aval25" value="')[1].split('"')[0] %}
        {{ val == "ZAL" }}
      {% else %}
        false
      {% endif %}
    device_class: power
    scan_interval: 30
```



do scripts.yaml
```yaml
zapisz_tryb_cwu:
  alias: Zapisz ustawienia CWU
  sequence:
    - service: rest_command.zapisz_cwu
  mode: single
zapisz_temperatury_cwu:
  alias: Zapisz temperatury CWU
  sequence:
    - service: rest_command.ustaw_temp_cwu
  mode: single
zapisz_ogrzewanie:
  alias: Zapisz tryb ogrzewania
  sequence:
    - service: rest_command.zapisz_tryb_ogrzewania
  mode: single
zapisz_temp_pomieszczenia:
  alias: Zapisz temperaturę pomieszczenia
  sequence:
    - service: rest_command.zapisz_temp_pomieszczenia
  mode: single
zapisz_antylegionella:
  alias: Zapisz tryb antylegionella
  sequence:
    - service: rest_command.zapisz_antylegionella
  mode: single
```
Karty sterowania w HA itp
```yaml
type: vertical-stack
cards:
  - type: entities
    title: Tryb pracy CWU
    entities:
      - input_select.tryb_cwu
      - entity: binary_sensor.stiebel_eltron_stan_cwu
        name: Aktualny stan CWU
        icon: |
          {% if is_state('binary_sensor.stiebel_eltron_stan_cwu', 'on') %}
            mdi:water-pump
          {% else %}
            mdi:water-pump-off
          {% endif %}
      - entity: sensor.temperatura_zasobnika_na_dole_stiebel_eltron
        name: Temperatura zbiornika CWU
  - type: button
    name: 💾 Zapisz ustawienia CWU
    tap_action:
      action: call-service
      service: script.zapisz_tryb_cwu
grid_options:
  columns: 12
  rows: auto
```

```yaml
type: vertical-stack
cards:
  - type: entities
    title: Ustawienia temperatur CWU
    entities:
      - input_number.temp_cwu_on
      - input_number.temp_cwu_off
  - type: button
    name: 💾 Zapisz temperatury CWU
    tap_action:
      action: call-service
      service: script.zapisz_temperatury_cwu

```

```yaml
type: vertical-stack
cards:
  - type: entities
    title: Tryb pracy ogrzewania
    entities:
      - input_select.tryb_ogrzewania
      - entity: binary_sensor.pompa_stiebel_eltron_stan_ogrzewania
        name: Aktualny stan ogrzewania
        icon: >
          {% if is_state('binary_sensor.pompa_stiebel_eltron_stan_ogrzewania',
          'on') %}
            mdi:radiator
          {% else %}
            mdi:radiator-off
          {% endif %}
  - type: button
    name: 🔥 Zapisz tryb ogrzewania
    tap_action:
      action: call-service
      service: script.zapisz_ogrzewanie
grid_options:
  columns: 12
  rows: auto

```

```yaml
type: vertical-stack
cards:
  - type: entities
    title: Temperatura pomieszczenia
    entities:
      - input_number.temp_pomieszczenia
  - type: button
    name: 💾 Zapisz temperaturę pomieszczenia
    tap_action:
      action: call-service
      service: script.zapisz_temp_pomieszczenia

```

```yaml
type: vertical-stack
cards:
  - type: entities
    title: Antylegionella
    entities:
      - input_select.antylegionella
      - entity: binary_sensor.stiebel_eltron_antylegionella_stan
        name: Aktualny stan
        icon: >
          {% if is_state('binary_sensor.stiebel_eltron_antylegionella_stan',
          'on') %}
            mdi:biohazard
          {% else %}
            mdi:biohazard-off
          {% endif %}
  - type: button
    name: 💾 Zapisz tryb Antylegionella
    tap_action:
      action: call-service
      service: script.zapisz_antylegionella

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




