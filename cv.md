
# [rsschool-cv](https://annavoloshina.github.io/rsschool-cv/)

## Alexander Hatkov
### PHP Developer

![cv](https://github.com/AleksanderKhatkov9/rsschool-cv/blob/gh-pages/sasha.jpeg)


### Contact information:
- **Phone:** +375 (33) 638-91-05
- **E-mail:** bendar01991@gmail.com
- **Telegram:** @SashsaHatkov
- [LinkedIn](https://www.linkedin.com/in/alexander-hatkov-645a171b1/)
- [Rabota](https://rabota.by/resume/9b11345aff08acc0730039ed1f70596a71454c)

---

### Summary:
I have experience in software development. My skills include writing web applications in Java, JavaScript, PHP, Python programming languages.

---

### Skills and Proficiency:
- Framework: Laravel
- Programming languages: PHP, JavaScript 
- Frontend: HTML, CSS, SASS, JS,  Vue.js 
- DBMS: MySQL, PostgreSQL 
- Operating systems: Linux, Docker, Vagrant (configuration and administration)
- Web servers: Apache, Nginx.
- Git, SVN

---

### Code example:
```javascript
import axios from "axios";
import Cookies from 'js-cookie';
import '../../../css/mao/map.css';

let myMap, objectManager;
let visibleObjects = []

export default {
    components: {Map},
    data() {
        return {
            yandexMap: null,
            objectManager: null,
            cities: [],
            clients: [],
            selectedCityId: null,
            type: [],
            selectedCity: null,
            elements: [],
            boards: null,
            inputValue: '',
        };
    },
    created() {
        this.loadMap();
        this.getCityData();
    },
    methods: {
        loadMap() {
            const url = "https://api-maps.yandex.ru/2.1/?lang=ru-RU&apikey=c1ab5aff-3d31-475b-9831-c89eec2a488e&suggest_apikey=71ee514b-612e-40b1-bd80-6706004c3ca1";
            const script = document.createElement("script");
            script.src = url;
            script.async = true;
            script.onload = () => this.initMap();
            document.body.appendChild(script);
        },
        initMap() {
            ymaps.ready(() => {
                myMap = new ymaps.Map("map", {
                        center: [53.906907, 27.557669],
                        zoom: 10,
                        controls: ["zoomControl", "geolocationControl", "routeButtonControl", "smallMapDefaultSet"],
                        searchControlProvider: 'yandex#search'
                    },
                    {
                        minZoom: 7,
                        maxZoom: 18,
                        geolocationControlFloat: "right",
                        zoomControlPosition: {
                            top: "50px",
                            right: "10px",
                        },
                        searchControlFloatIndex: "1000",
                        restrictMapArea: [
                            [50.5, 17],
                            [57, 39],
                        ],
                    }
                );

                this.getMapData();
            });
        },
        getMapData() {
            axios
                .get("/api/json")
                .then((resp) => {
                    if (resp && resp.data && resp.data.length > 0) {
                        const features = resp.data;
                        console.log(features);
                        this.createObjectManager(features);
                    } else {
                        console.error("Ошибка: возвращенные данные недоступны или пусты.");
                    }
                })
                .catch((err) => {
                    console.error("Ошибка при загрузке данных:", err);
                });
        },

        onObjectEvent(e) {
            var objectId = e.get('objectId');
            if (e.get('type') === 'mouseenter') {
                // Метод setObjectOptions позволяет задавать опции объекта "на лету".
                objectManager.objects.setObjectOptions(objectId, {
                    preset: 'islands#yellowIcon'
                });
            } else {
                objectManager.objects.setObjectOptions(objectId, {
                    preset: 'islands#blueIcon'
                });
            }
        },

        createObjectManager(features) {
            objectManager = new ymaps.ObjectManager({
                clusterize: true,
                geoObjectOpenBalloonOnClick: true,
                clusterOpenBalloonOnClick: true,
                clusterDisableClickZoom: true,
                clusterHideIconOnBalloonOpen: false,
                groupByCoordinates: false,
                clusterBalloonContentLayout: ymaps.templateLayoutFactory.createClass([
                    '{% for geoObject in properties.geoObjects %}',
                    '{{ geoObject.properties.balloonContent|raw }}',
                    '{% endfor %}'
                ].join('')),
            });
            let featureMap = {};
            features.forEach(function (feature) {
                featureMap[feature.properties.id] = feature;
                objectManager.add({
                    type: 'Feature',
                    id: feature.properties.id,
                    geometry: {
                        type: 'Point',
                        coordinates: feature.geometry.coordinates,
                    },
                    properties: {
                        balloonContent: `
                        <div class="feature1">
                        <li class="feature" data-feature="${feature.properties.id}">
                            <ul class="f-info">
                                <li class="f-head">
                                    <span class="f-type">${feature.properties.type}</span>
                                    <span class="f-size">${feature.properties.size}</span>
                                    <span class="f-side">${feature.properties.side}</span>
                                    <img class="f-light" src="img/light_${feature.properties.light === 1 ? 'on' : 'off'}.png" alt="light">
                                </li>
                                <li class="f-location">
                                    <span class="f-city">${feature.properties.city}</span>, <span class="f-address">${feature.properties.address}</span>
                                </li>
                                <li>
                                     <input type="file" id="file" name="files[]" accept="image/png, image/jpeg" class="selectFile" multiple v-model="file"/>
                                     <button type="button" class="sendFile" style="margin-left: 10px">Отправить</button>
                                </li>
                            </ul>
                            <div class="f-img"><img src="${feature.properties.img}" alt="${feature.properties.address}"/></div>
                        </li>
                     </div>
                `
                    },
                    options: {
                        iconLayout: 'default#image',
                        iconImageHref: feature.properties.img === 'img/noimg.png' ? feature.properties.icon_not_selected : feature.properties.icon,
                        iconImageSize: [30, 30],
                        balloonCloseButton: true,
                        hideIconOnBalloonOpen: false,
                        preset: feature.properties.img === 'img/noimg.png' ? 'islands#redClusterIcons' : 'islands#blueClusterIcons',
                    }
                })
            });

            this.addButtonToMap(myMap);
            document.addEventListener('click', (event) => {
                if (event.target.matches('.sendFile')) {
                    let featureId = event.target.closest('.feature').dataset.feature;
                    let fileInput = event.target.closest('.feature').querySelector('input[type="file"]').files[0];
                    this.sendFileAjax(fileInput, featureMap[featureId], myMap);
                }
            });

            let debounceTimer;
            myMap.events.add(['boundschange'], (e) => {
                let minZoom = 18;
                if (e.get('oldZoom') !== e.get('newZoom')) {
                    if (e.get('newZoom') === minZoom) {
                        clearTimeout(debounceTimer);
                        debounceTimer = setTimeout(() => {
                            myMap.geoObjects.each((geoObject) => {
                                myMap.geoObjects.remove(geoObject);
                            });
                            const objectsByBoardId = {};
                            // Группируем объекты по board_id
                            features.forEach(function (feature) {
                                const boardId = feature.properties.board_id;
                                if (!objectsByBoardId[boardId]) {
                                    objectsByBoardId[boardId] = [];
                                }
                                objectsByBoardId[boardId].push(feature);
                            });
                            // Создаем балун для объектов с одинаковым board_id
                            for (const boardId in objectsByBoardId) {
                                const objects = objectsByBoardId[boardId];
                                let balloonContent = '';
                                objects.forEach(function (feature) {
                                    balloonContent += `
                                <div class="feature1">
                                    <li class="feature" data-feature="${feature.properties.id}">
                                        <ul class="f-info">
                                            <li class="f-head">
                                                <span class="f-type">${feature.properties.type}</span>
                                                <span class="f-size">${feature.properties.size}</span>
                                                <span class="f-side">${feature.properties.side}</span>
                                                <img class="f-light" src="img/light_${feature.properties.light === 1 ? 'on' : 'off'}.png" alt="light">
                                            </li>
                                            <li class="f-location">
                                                <span class="f-city">${feature.properties.city}</span>, <span class="f-address">${feature.properties.address}</span>
                                            </li>
                                            <li>
                                                <input type="file" id="file" name="files[]" accept="image/png, image/jpeg" class="selectFile" multiple v-model="file"/>
                                                <button type="button" class="sendFile" style="margin-left: 10px">Отправить</button>
                                            </li>
                                        </ul>
                                        <div class="f-img"><img src="${feature.properties.img}" alt="${feature.properties.address}"/></div>
                                    </li>
                                </div>`;

                                    const myPlacemark = new ymaps.Placemark(objects[0].geometry.coordinates, {
                                        balloonContent: balloonContent
                                    }, {
                                        iconLayout: 'default#image',
                                        iconImageHref: feature.properties.img === 'img/noimg.png' ? feature.properties.icon_not_selected : feature.properties.icon,
                                        iconImageSize: [30, 30],
                                        balloonCloseButton: true,
                                    });
                                    myMap.geoObjects.add(myPlacemark);
                                });
                            }
                        }, 250);

                    } else if (e.get('newZoom') === 12) {
                        clearTimeout(debounceTimer);
                        debounceTimer = setTimeout(() => {
                            myMap.geoObjects.removeAll();
                            objectManager.clusters.each(function (cluster) {
                                let objects = cluster.features
                                if (objects[0].properties.board_id && objects.every(o => o.properties.board_id === objects[0].properties.board_id)) {
                                    cluster.options.preset = 'islands#default'
                                    cluster.options.clusterIconContentLayout = null
                                    cluster.options.clusterIcons = [
                                        {
                                            href: objects[0].properties.icon,
                                            size: [30, 30],
                                            offset: [-15, -15],
                                        }
                                    ]
                                }
                            });
                            myMap.geoObjects.add(objectManager)
                        }, 250);
                    }
                }
            });
            myMap.geoObjects.add(objectManager);

            setTimeout(this.updateCenter, 100);
            objectManager.clusters.events.add(['mouseenter', 'mouseleave'], this.onClusterEvent);
        },
