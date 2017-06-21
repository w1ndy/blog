---
title: Gallery
date: 2017-06-21 19:41:01
comments: false
---

{% raw %}
<style>
.gallery {
    margin-top: 40px;
    display: flex;
    flex-wrap: wrap;
}
.image-container {
    flex: 0 0 33%;
}
.image {
    margin: 5px;
    padding-top: 100%;
    background-size: cover;
    border: 1px solid #ddd;
    background-position: center;
}
.image:hover {
    cursor: pointer;
    border: 1px solid #999;
}
.pagination-cont {
    width: fit-content;
    margin: 40px auto;
}
.page-button {
    width: 30px;
    line-height: 30px;
    font-size: 14px;
    border: 1px solid #ddd;
    display: inline-block;
    text-align: center;
    margin: 0 2px;
    user-select: none;
    cursor: default;
}
.page-button.disabled {
    visibility: hidden;
}
.page-button.active, .page-button:hover {
    background-color: #eee;
}
</style>
<div class="gallery" id="gallery">
</div>
<div class="pagination-cont">
    <span class="prev page-button" id="prev-page">&laquo;</span>
    <span class="gallery-pagination" id="pagination"></span>
    <span class="next page-button" id="next-page">&raquo;</span>
</div>
<script>
(function () {
    let imageUrls = [], currentPage = 1, totalPage = 1;
    const ImagesPerPage = 18;
    const gallery = document.getElementById('gallery');
    const prevButton = document.getElementById('prev-page');
    const nextButton = document.getElementById('next-page');
    const pagination = document.getElementById('pagination');
    const createImageContainer = url => {
        const container = document.createElement('div');
        const image = document.createElement('div');
        container.className = 'image-container';
        image.className = 'image';
        image.style.backgroundImage = `url(${url.url}-thumbnail)`;
        image.onclick = () => {
            window.open(url.ins, 'Instagram');
        };
        container.appendChild(image);
        gallery.appendChild(container);
    };
    const toggleElementState = (elem, state) => {
        if (state) {
            elem.classList.remove('disabled');
        } else {
            elem.classList.add('disabled');
        }
    };
    const removeAllChildElements = elem => {
        while (elem.firstChild) {
            elem.removeChild(elem.firstChild);
        }
    };
    const initializeGalleryPage = () => {
        removeAllChildElements(gallery);
        imageUrls.slice(
            (currentPage - 1) * ImagesPerPage,
            currentPage * ImagesPerPage)
        .forEach(createImageContainer);
    };
    const createPageButton = pageNo => {
        const button = document.createElement('span');
        button.className = 'page-button'
            + (pageNo == currentPage ? ' active' : '');
        button.innerHTML = pageNo.toString();
        button.onclick = () => {
            currentPage = pageNo;
            initializePaginationButtons();
        };
        pagination.appendChild(button);
    };
    const initializePaginationButtons = () => {
        removeAllChildElements(pagination);
        toggleElementState(prevButton, currentPage != 1);
        toggleElementState(nextButton, currentPage != totalPage);
        for (let k = 1; k <= totalPage; k++) {
            createPageButton(k);
        }
        initializeGalleryPage();
        document.body.scrollTop = 0;
    };
    fetch('http://orwaz3d8t.bkt.clouddn.com/images.json?v=1')
    .then(resp => resp.json())
    .then(images => {
        imageUrls = images.urls;
        totalPage = Math.ceil(imageUrls.length / ImagesPerPage);
        prevButton.onclick = () => {
            currentPage -= 1;
            initializePaginationButtons();
        };
        nextButton.onclick = () => {
            currentPage += 1;
            initializePaginationButtons();
        };
        initializePaginationButtons();
    });
}())
</script>
{% endraw %}