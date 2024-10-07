<template>
    <div>
      <h1>Danh sách sinh viên</h1>
      <button @click="showCreateForm">Thêm sinh viên</button>
      <table>
        <thead>
          <tr>
            <th>Id</th>
            <th>Tên</th>
            <th>Email</th>
            <th>Địa chỉ</th>
            <th>Số điện thoại</th>
            <th>Trạng thái</th>
            <th>Ngày tạo</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(student, index) in students" :key="student.id">
            <td>{{ index + 1 }}</td>
            <td>{{ student.student_name }}</td>
            <td>{{ student.email }}</td>
            <td>{{ student.address }}</td>
            <td>{{ student.phone }}</td>
            <td>{{ student.status ? 'Hoạt động' : 'Đình chỉ' }}</td>
            <td>{{ student.created_at }}</td>
            <td>
              <button @click="editStudent(student)">Sửa</button>
              <button @click="removeStudentById(student.id)">Xóa</button>
            </td>
          </tr>
        </tbody>
      </table>
  
      <div v-if="isFormVisible" class="student-form">
        <h2>{{ isEditMode ? 'Cập nhật sinh viên' : 'Thêm sinh viên mới' }}</h2>
        <label>Tên sinh viên</label>
        <input v-model="selectedStudent.student_name" type="text" />
  
        <label>Email</label>
        <input v-model="selectedStudent.email" type="text" />
  
        <label>Địa chỉ</label>
        <input v-model="selectedStudent.address" type="text" />
  
        <label>Số điện thoại</label>
        <input v-model="selectedStudent.phone" type="text" />
  
        <label>Trạng thái</label>
        <input v-model="selectedStudent.status" type="checkbox" />
  
        <button @click="cancelForm">Hủy</button>
        <button @click="isEditMode ? confirmUpdate() : createStudent()">Lưu</button>
      </div>
    </div>
  </template>
  <script setup>
  import { ref, onMounted } from 'vue'
  import Swal from 'sweetalert2'
  import axios from 'axios'
  
  const students = ref([])
  
  const fetchData = async () => {
    try {
      const response = await axios.get('http://localhost:3000/students')
      students.value = response.data
    } catch (error) {
      console.error('Error fetching data:', error)
    }
  }
  
  const isEditMode = ref(false)
  const isFormVisible = ref(false)
  const selectedStudent = ref({
    student_name: '',
    email: '',
    address: '',
    phone: '',
    status: false,
    created_at: ''
  })
  
  const showCreateForm = () => {
    isEditMode.value = false
    isFormVisible.value = true
    selectedStudent.value = { student_name: '', email: '', address: '', phone: '', status: false, created_at: '' }
  }
  
  const cancelForm = () => {
    isFormVisible.value = false
  }
  
  const createStudent = () => {
    if (!selectedStudent.value.student_name || !selectedStudent.value.email || !selectedStudent.value.address || !selectedStudent.value.phone) {
      Swal.fire('Lỗi', 'Vui lòng điền đầy đủ thông tin', 'error')
      return
    }
  
    Swal.fire({
      title: 'Xác nhận',
      text: 'Bạn có chắc chắn muốn thêm sinh viên này?',
      icon: 'warning',
      showCancelButton: true,
      confirmButtonText: 'Thêm',
      cancelButtonText: 'Hủy',
    }).then(async (result) => {
      if (result.isConfirmed) {
        const newStudent = { ...selectedStudent.value, id: Date.now().toString(), created_at: new Date().toISOString() }
  
        try {
          await axios.post('http://localhost:3000/students', newStudent)
          Swal.fire('Thành công', 'Sinh viên đã được thêm', 'success')
          isFormVisible.value = false
          fetchData() // Lấy lại dữ liệu mới nhất
        } catch (error) {
          console.error('Error adding student:', error)
        }
      }
    })
  }
  
  const editStudent = (student) => {
    selectedStudent.value = { ...student }
    isEditMode.value = true
    isFormVisible.value = true
  }
  
  const confirmUpdate = () => {
    if (!selectedStudent.value.student_name || !selectedStudent.value.email || !selectedStudent.value.address || !selectedStudent.value.phone) {
      Swal.fire('Lỗi', 'Vui lòng điền đầy đủ thông tin', 'error')
      return
    }
  
    Swal.fire({
      title: 'Xác nhận',
      text: 'Bạn có chắc chắn muốn cập nhật sinh viên này?',
      icon: 'warning',
      showCancelButton: true,
      confirmButtonText: 'Cập nhật',
      cancelButtonText: 'Hủy',
    }).then(async (result) => {
      if (result.isConfirmed) {
        try {
          await axios.put(`http://localhost:3000/students/${selectedStudent.value.id}`, selectedStudent.value)
          Swal.fire('Thành công', 'Sinh viên đã được cập nhật', 'success')
          isFormVisible.value = false
          fetchData() // Lấy lại dữ liệu mới nhất
        } catch (error) {
          console.error('Error updating student:', error)
        }
      }
    })
  }
  
  const removeStudentById = (id) => {
    Swal.fire({
      title: 'Xác nhận',
      text: 'Bạn có chắc chắn muốn xóa sinh viên này?',
      icon: 'warning',
      showCancelButton: true,
      confirmButtonText: 'Xóa',
      cancelButtonText: 'Hủy',
    }).then(async (result) => {
      if (result.isConfirmed) {
        try {
          await axios.delete(`http://localhost:3000/students/${id}`)
          Swal.fire('Thành công', 'Sinh viên đã được xóa', 'success')
          fetchData() // Lấy lại dữ liệu mới nhất
        } catch (error) {
          console.error('Error deleting student:', error)
        }
      }
    })
  }
  
  onMounted(() => {
    fetchData()
  })
  </script>
<style>
img {
  display: block;
  margin-bottom: 10px;
}
li {
  margin-bottom: 20px;
  list-style-type: none;
}
button {
  margin-top: 10px;
  padding: 5px 10px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
button:hover {
  background-color: #0056b3;
}
</style>
    