<template>
    <Suspense>
        <HeaderComponent text-color="white"></HeaderComponent>
        <template #fallback>

        </template>
    </Suspense>
    <Suspense>
        <SidebarComponent text-color="white"></SidebarComponent>
    </Suspense>

    <div id="uploadPost-section" class="d-flex p-lg-5" style="background-color: white;">
        <div id="choosePost" class="btn-group btn-group-vertical justify-content-start d-flex flex-column sticky-top"
            role="group" aria-label="Basic radio toggle button group">
            <div class="w-100" style="margin-bottom: 20px;">
                <input @click="clearForm()" type="radio" class="btn-check" name="btnradio" id="clear" autocomplete="off"
                    :value="0" checked>
                <label class="btn btn-outline-secondary w-100" for="clear">Mới</label>
            </div>
            <h5 class="text-center" v-if="posts.length > 0">Các bài viết đang được chỉnh sửa</h5>
            <div v-for="(post, index) in posts" :key="post.id" class="w-100">
                <input @click="changeForm(post.id); choosenCreatedPostIndex = index; choosenCreatedPostId = post.id"
                    type="radio" class="btn-check" name="btnradio" :id="'btnradio' + post.id" autocomplete="off"
                    :value="post.id">
                <label class="btn btn-outline-dark w-100" :for="'btnradio' + post.id">{{ post.title }}</label>
            </div>

            <h5 class="text-primary" style="margin-top: 20px;" v-if="postsReviewing.length > 0">Bài viết chờ
                duyệt </h5>
            <div v-for="(post, index) in postsReviewing" :key="post.id" class="w-100" style="margin-bottom: 5px">
                <input @click="changeForm(post.id); choosenCreatedPostIndex = index; choosenCreatedPostId = post.id"
                    type="radio" class="btn-check" name="btnradio" :id="'btnradio' + post.id" autocomplete="off"
                    :value="post.id">
                <label class="btn btn-outline-primary w-100" :for="'btnradio' + post.id">{{ post.title }}</label>
            </div>

            <h5 class="text-danger" style="margin-top: 20px;" v-if="postsRejected.length > 0">Bài viết bị từ chối</h5>
            <div v-for="(post, index) in postsRejected" :key="post.id" class="w-100" style="margin-bottom: 5px">
                <input @click="changeForm(post.id); choosenCreatedPostIndex = index; choosenCreatedPostId = post.id"
                    type="radio" class="btn-check" name="btnradio" :id="'btnradio' + post.id" autocomplete="off"
                    :value="post.id">
                <label class="btn btn-outline-danger w-100" :for="'btnradio' + post.id">{{ post.title }}</label>
                <div class="text-danger">Lý do từ chối: {{ post.rejectMessage }}</div>
            </div>
        </div>

        <form style="width: 70vw;">

            <label for="selectCategory">Thể loại của bài viết</label>
            <select v-model="editPostForm.categoryId" id="selectCategory" class="form-select"
                aria-label="Default select example" required>
                <option v-for="cate in categories" :key="cate.id" :value="cate.id">
                    <span>{{ cate.name }}</span>
                </option>
            </select>

            <div>
                <div v-if="filterTagsByCategoryId().length > 0">Nhãn bài viết</div>
                <div class="btn-group" style="flex-wrap: wrap;" role="group"
                    aria-label="Basic checkbox toggle button group">
                    <div v-for="(tag) in filterTagsByCategoryId()" :key="tag.id">
                        <div v-if="tag.isLock == true || (tag.isLock == false && tag.createdById == currentUser.id)"  style="margin-right: 5px; margin-bottom: 5px;">
                            <input v-model="trackingTags[tag.id]" type="checkbox" class="btn-check"
                                :id="'btncheck' + tag.id" autocomplete="off">
                            <label class="btn btn-outline-secondary" :for="'btncheck' + tag.id">{{ tag.name }}</label>
                        </div>
                    </div>
                </div>
                <button type="button" :disabled="editPostForm.categoryId == 0" class="btn btn-light"
                    style="width: 50px; margin-bottom: 30px;" data-bs-toggle="modal" data-bs-target="#addTag">
                    <i class="fa-solid fa-plus"></i>
                </button>

            </div>
            <div class="d-flex">
                <div class="w-50">
                    <div>Nguồn bài viết (Nếu có)</div>
                    <select v-model="editPostForm.contentSourceId"  class="form-select" aria-label="Default select example">
                        <option v-for="cs in contentSources" :key="cs.id" :value="cs.id" :style="{'background-image': 'url(' + cs.avatar + ')'}">
                            <span>{{ cs.name }}</span>
                        </option>
                    </select>
                </div>
                <div class='w-50'>
                    <label for="originalLink">Link bài viết gốc (Nếu có)</label>
                    <input style="margin-bottom: 20px;" v-model="editPostForm.originalLink" id="originalLink" type="text" class="form-control" >
                </div>
            </div>

            <label for="titlePost" style="font-size: 30px;">Tiêu đề bài viết</label>
            <input v-model="editPostForm.title" id="titlePost" type="text" class="form-control" required>

            <div class="mb-3 text-center d-flex flex-column justify-content-center" style="margin: 50px 0 50px 0">
                <img v-if="editPostForm.imageCover != null && editPostForm.imageCover[0] == 'f'"
                    style="margin-bottom: 50px;" class="img-thumbnail" width="500px"
                    :src="'http://localhost:8080' + editPostForm.imageCover.replace('files', '')" alt="">

                <label for="formFile" class="form-label">File ảnh bìa bài viết</label>
                <input @change="previewFile($event)" class="form-control" type="file" id="formFile">
            </div>

            <div id="quill">
                <div id="quillEditor">
                    Nội dung bài viết:
                    <QuillEditor v-model:content="content" contentType="html" toolbar="full"></QuillEditor>

                </div>
            </div>
            <div class="d-flex justify-content-end" v-if="isLogin">
                <button class="btn-upload-post m-1 btn btn-danger" @click="updatePost($event, 'reviewing')"
                    style="padding: 15px;">
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor"
                        class="bi bi-upload" viewBox="0 0 16 16">
                        <path
                            d="M.5 9.9a.5.5 0 0 1 .5.5v2.5a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1v-2.5a.5.5 0 0 1 1 0v2.5a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2v-2.5a.5.5 0 0 1 .5-.5" />
                        <path
                            d="M7.646 1.146a.5.5 0 0 1 .708 0l3 3a.5.5 0 0 1-.708.708L8.5 2.707V11.5a.5.5 0 0 1-1 0V2.707L5.354 4.854a.5.5 0 1 1-.708-.708z" />
                    </svg>
                    Yêu cầu xuất bản
                </button>
                <button class="btn-upload-post m-1 btn btn-primary" @click="updatePost($event, 'created')"
                    style="padding: 15px;">
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor"
                        class="bi bi-file-earmark-check-fill" viewBox="0 0 16 16">
                        <path
                            d="M9.293 0H4a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h8a2 2 0 0 0 2-2V4.707A1 1 0 0 0 13.707 4L10 .293A1 1 0 0 0 9.293 0M9.5 3.5v-2l3 3h-2a1 1 0 0 1-1-1m1.354 4.354-3 3a.5.5 0 0 1-.708 0l-1.5-1.5a.5.5 0 1 1 .708-.708L7.5 9.793l2.646-2.647a.5.5 0 0 1 .708.708" />
                    </svg>
                    Lưu
                </button>
                <button class="btn-upload-post m-1 btn btn-secondary" @click="removePost()" style="padding: 15px;">
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor"
                        class="bi bi-trash" viewBox="0 0 16 16">
                        <path
                            d="M5.5 5.5A.5.5 0 0 1 6 6v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5m2.5 0a.5.5 0 0 1 .5.5v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5m3 .5a.5.5 0 0 0-1 0v6a.5.5 0 0 0 1 0z" />
                        <path
                            d="M14.5 3a1 1 0 0 1-1 1H13v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V4h-.5a1 1 0 0 1-1-1V2a1 1 0 0 1 1-1H6a1 1 0 0 1 1-1h2a1 1 0 0 1 1 1h3.5a1 1 0 0 1 1 1zM4.118 4 4 4.059V13a1 1 0 0 0 1 1h6a1 1 0 0 0 1-1V4.059L11.882 4zM2.5 3h11V2h-11z" />
                    </svg>
                    Xóa
                </button>
            </div>
        </form>
        <div class="modal fade" id="addTag" tabindex="-1" aria-labelledby="addTagLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="">Thêm nhãn dán</h5>
                    </div>
                    <div class="modal-body">
                        <input type="text" class="form-control" v-model="newTagName" required>
                    </div>
                    <div class="modal-footer">
                        <button :disabled="newTagName == ''" type="button" class="btn btn-primary" data-bs-dismiss="modal"
                            @click="addTag">Thêm</button>
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Hủy</button>
                    </div>
                </div>
            </div>
        </div>

    </div>
</template>
<script setup lang="ts">
import HeaderComponent from '@/components/HeaderComponent.vue';
//@ts-ignore
import SidebarComponent from '@/components/SidebarComponent.vue';
import contentSourcesService from '@/services/contentSources.service';
import categoriesService from '@/services/categories.service';
import postsService from '@/services/posts.service';
import tagsService from '@/services/tags.service';
import { onMounted, ref } from 'vue';
import { QuillEditor } from '@vueup/vue-quill';
import checkLogin from '@/utilities/utilities';
import { useCookies } from 'vue3-cookies';
import { toast } from 'vue3-toastify';
import {useRouter } from 'vue-router'

const router = useRouter()
const categories = ref([
    {
        id: '',
        createdAt: "",
        updatedAt: "",
        deletedAt: null,
        name: '',
        imageURL: null
    }
])

const fileImage = ref({})
const content = ref('')
const tags = ref([
    {
        id: 0,
        createdAt: "",
        updatedAt: "",
        deletedAt: null,
        name: "",
        categoryId: 0,
        createdById: 0,
        isLock: false,
        posts: [
            {
                id: '',
                createdAt: "",
                updatedAt: "",
                deletedAt: null,
                title: "",
                content: "",
                sharePostId: null,
                originalPostURL: "",
                publishDate: "",
                imageURL: "",
                status: "",
                type: "",
                readTime: '',
                totalLike: '',
                totalDislike: '',
                totalShare: '',
                categoryId: '',
                createdById: '',
                contentSourceId: ''
            }
        ],
        category: {
            id: 0,
            createdAt: "",
            updatedAt: "",
            deletedAt: null,
            name: "",
            imageURL: null
        }
    }
])

const contentSources = ref([
    {
        id: '',
        createdAt: "",
        updatedAt: "",
        deletedAt: null,
        name: "",
        avatar: "",
        posts: [
            {
                id: '',
                createdAt: "",
                updatedAt: "",
                deletedAt: null,
                title: "",
                content: "",
                sharePostId: null,
                originalPostURL: "",
                publishDate: "",
                imageURL: "",
                status: "",
                type: "",
                readTime: '',
                totalLike: '',
                totalDislike: '',
                totalShare: '',
                categoryId: '',
                createdById: '',
                contentSourceId: ''
            }
        ]
    }
])

type postType = {
    rejectMessage: '',
    id: 0,
    createdAt: "",
    updatedAt: "",
    deletedAt: null,
    title: "",
    content: "",
    sharePostId: null,
    originalPostURL: "",
    publishDate: "",
    imageURL: "",
    status: "",
    type: "",
    readTime: 0,
    totalLike: 0,
    totalDislike: 0,
    totalShare: 0,
    categoryId: 0,
    createdById: 0,
    contentSourceId: 0
    tags: [
        {
            id: 0,
            createdAt: "",
            updatedAt: "",
            deletedAt: null,
            name: "",
            categoryId: 0,
            createdById: 0,
            isLock: false
        }
    ]
}

const posts = ref([{} as postType])
const postsReviewing = ref([{} as postType])
const postsRejected = ref([{} as postType])
const postAllUser = ref([{} as postType])

const choosenCreatedPostId = ref(0)
const choosenCreatedPostIndex = ref(0)

const editPostForm = ref({
    postId: 0,
    categoryId: 0,
    tag: [] as string[],
    contentSourceId: 0,
    title: '',
    imageCover: '',
    content: '',
    originalLink: '',
})

const currentUser = ref({
    id: 0,
    createdAt: "",
    updatedAt: "",
    deletedAt: null,
    email: "",
    username: "",
    firstName: "",
    lastName: "",
    fullName: "",
    about: null,
    youtubeLink: null,
    facebookLink: null,
    linkedinLink: null,
    twitterLink: null,
    totalFollower: 0,
    totalFollowee: 0,
    refreshToken: null,
    phoneNumber: "",
    birthday: "",
    avatar: "",
    role: "",
    categories: [{
        id: 0,
        createdAt: "",
        updatedAt: "",
        deletedAt: null,
        name: "",
        imageURL: null
    }]
})
const isLogin = ref(false)
const cookies = useCookies();
const tokenBearer = cookies.cookies.get('Token');
const trackingTags = ref({} as Record<number, boolean>)

try {
    currentUser.value = await checkLogin();
    if (currentUser.value !== null && currentUser.value['id'] !== null) {
        isLogin.value = true;
    }
} catch (err) {
    console.log(err)
}

async function changeForm(id: number) {
    postAllUser.value.forEach((post) => {
        if (post.id == id) {
            editPostForm.value.postId = post.id
            editPostForm.value.categoryId = post.categoryId
            editPostForm.value.title = post.title
            editPostForm.value.contentSourceId = post.contentSourceId
            fileImage.value = post.imageURL
            editPostForm.value.imageCover = post.imageURL
            editPostForm.value.content = post.content
            editPostForm.value.originalLink = post.originalPostURL
            content.value = post.content
            for (let i = 0; i < filterTagsByCategoryId().length; i++) {
                trackingTags.value[filterTagsByCategoryId()[i].id] = false
            }

            post.tags.forEach(tag => {
                for (let j = 0; j < filterTagsByCategoryId().length; j++) {
                    if (filterTagsByCategoryId()[j].id == tag.id) {
                        trackingTags.value[filterTagsByCategoryId()[j].id] = true
                    }
                }
            });
        }
    })
}

function previewFile(e: any) {
    fileImage.value = e.target.files[0]
}

function clearForm() {
    for (let index in trackingTags.value) {
        trackingTags.value[index] = false
    }
    content.value = ''
    choosenCreatedPostId.value = 0
    editPostForm.value.categoryId = 0
    editPostForm.value.title = ''
    editPostForm.value.contentSourceId = 0
    editPostForm.value.imageCover = 'https://mdbootstrap.com/img/Photos/Others/placeholder.jpg'
    editPostForm.value.content = ''
    editPostForm.value.originalLink = ''
}

function filterTagsByCategoryId() {
    const tagsTemp = tags.value.filter((tag) => tag.categoryId == editPostForm.value.categoryId)
    
    return tagsTemp
}

async function updatePost(e: any, status: string) {
    e.preventDefault();
    try {
        if (content.value == '') {
            toast.warning("Vui lòng nhập nội dung", {
                autoClose: 2000,
            })
            throw new Error('Thiếu nội dung!')
        }
        if (editPostForm.value.contentSourceId != 0 && editPostForm.value.originalLink != null && editPostForm.value.originalLink.length == 0) {
            toast.warning("Vui lòng nhập đường dẫn phù hợp với nguồn", {
                autoClose: 2000,
            })
            throw new Error('Thiếu đường dẫn!')
        }
        let tagsName = [] as string[]
        let cateTags = filterTagsByCategoryId()
        for (let index in trackingTags.value) {
            if (trackingTags.value[index]) {
                let choosenTag = cateTags.filter((tag) => tag.id == Number(index))
                tagsName.push(choosenTag[0].name)
            }
        }


        let type = 'internal_post'
        if (editPostForm.value.originalLink != null && editPostForm.value.originalLink.length > 0 ){
            type = 'external_personal_blog'
        }
        console.log(type)
        if (choosenCreatedPostId.value == 0) {
            await postsService.create(editPostForm.value.categoryId, tagsName, editPostForm.value.contentSourceId, editPostForm.value.title, content.value, editPostForm.value.originalLink, status, type, fileImage.value, tokenBearer)
            toast.success("Đã tạo bài viết thành công", {
                autoClose: 2000,
            })
        }
        else {
            await postsService.update(editPostForm.value.postId, editPostForm.value.categoryId, tagsName, editPostForm.value.contentSourceId, editPostForm.value.title, content.value, editPostForm.value.originalLink, status, type, fileImage.value, tokenBearer)
            toast.success("Đã cập nhật bài viết thành công", {
                autoClose: 2000,
            })
        }

        window.location.reload();
    } catch (err) {
        console.log(err);
    }
}

async function removePost() {
    try {
        await postsService.delete(choosenCreatedPostId.value, tokenBearer)
        posts.value.splice(choosenCreatedPostIndex.value, 1)
    } catch (error) {
        console.log(error)
    }
}

const newTagName = ref('')

async function addTag() {
    try {
        let resp = await tagsService.create({
            tags: [
                newTagName.value
            ],
            categoryId: editPostForm.value.categoryId
        }, tokenBearer);
        toast.success('Đã thêm thành công!', {
            autoClose: 1000
        })
        console.log(resp[0])
        tags.value.push(resp[0])
    } catch (error) {
        console.log(error)
    }
}

onMounted(async () => {
    try {
        if (isLogin.value == false){
            router.push({name: "home"})
        }
        // Post all user
        let pAll = await postsService.getAllForUser(currentUser.value.id);
        postAllUser.value = pAll.data

        // Post
        let p = await postsService.getAllStatusForUser(currentUser.value.id, 'created');
        posts.value = p.data

        clearForm()

        // Post reviewing
        let pR = await postsService.getAllStatusForUser(currentUser.value.id, 'reviewing');
        postsReviewing.value = pR.data

        // Post rejected
        let pRejected = await postsService.getAllStatusForUser(currentUser.value.id, 'reject');
        postsRejected.value = pRejected.data

        // categories
        let ts = await tagsService.getAll();
        let cats = await categoriesService.getAll();
        let cs = await contentSourcesService.getAll();

        tags.value = ts.data
        categories.value = cats.data
        contentSources.value = cs.data
    } catch (err) {
        console.log(err);
    }
})

</script>
<style>
.form-select {
    margin-bottom: 40px;
}

#quill {
    height: 550px;
}

#quillEditor {
    height: 450px;

}

#choosePost {
    margin-right: 50px;
    width: 13vw;
    height: 50vh;
}

@media only screen and (max-width: 650px) {
    #uploadPost-section {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    #quill {
        height: 350px;
    }

    #quillEditor {
        height: 220px;

    }

    #choosePost {
        position: static;
        margin-top: 20px;
        margin-right: 0;
        width: 80%;
        height: auto;
    }

    .btn-upload-post {
        font-size: small;
    }
}

@media only screen and (max-width: 490px) {
    #quill {
        height: 500px;
    }

    #quillEditor {
        height: 220px;

    }

    .btn-upload-post {
        font-size: xx-small;
    }
}
</style>