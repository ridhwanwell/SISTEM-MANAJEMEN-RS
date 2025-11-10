# SISTEM-MANAJEMEN-RS
import React, { useState, useEffect } from 'react';
import { Camera, Plus, Search, X, CheckCircle, AlertCircle } from 'lucide-react';

const App = () => {
  const [view, setView] = useState('home');
  const [equipment, setEquipment] = useState([]);
  const [selectedEquipment, setSelectedEquipment] = useState(null);
  const [showModal, setShowModal] = useState(false);
  const [modalType, setModalType] = useState('');
  const [searchTerm, setSearchTerm] = useState('');
  const [notification, setNotification] = useState(null);

  // Load data from storage
  useEffect(() => {
    loadEquipment();
  }, []);

  const loadEquipment = async () => {
    try {
      const result = await window.storage.get('hospital_equipment');
      if (result) {
        setEquipment(JSON.parse(result.value));
      }
    } catch (error) {
      console.log('No existing data');
    }
  };

  const saveEquipment = async (data) => {
    try {
      await window.storage.set('hospital_equipment', JSON.stringify(data));
    } catch (error) {
      console.error('Error saving:', error);
    }
  };

  const addEquipment = async (formData) => {
    const newEquipment = {
      id: Date.now().toString(),
      ...formData,
      records: [],
      createdAt: new Date().toISOString()
    };
    const updated = [...equipment, newEquipment];
    setEquipment(updated);
    await saveEquipment(updated);
    showNotification('Alat berhasil ditambahkan', 'success');
  };

  const addRecord = async (equipmentId, recordData) => {
    const updated = equipment.map(eq => {
      if (eq.id === equipmentId) {
        return {
          ...eq,
          records: [...eq.records, {
            id: Date.now().toString(),
            ...recordData,
            timestamp: new Date().toISOString()
          }]
        };
      }
      return eq;
    });
    setEquipment(updated);
    await saveEquipment(updated);
    
    if (recordData.type === 'laporan') {
      showNotification('Laporan telah dikirim ke PIC Elektromedis', 'success');
    } else {
      showNotification('Data berhasil disimpan', 'success');
    }
  };

  const showNotification = (message, type) => {
    setNotification({ message, type });
    setTimeout(() => setNotification(null), 3000);
  };

  const generateQRData = (eq) => {
    return `EQUIP-${eq.id}`;
  };

  const AddEquipmentForm = ({ onSubmit, onClose }) => {
    const [formData, setFormData] = useState({
      namaAlat: '',
      typeAlat: '',
      snAlat: '',
      ruangan: ''
    });

    const handleSubmit = () => {
      if (formData.namaAlat && formData.typeAlat && formData.snAlat && formData.ruangan) {
        onSubmit(formData);
        onClose();
      }
    };

    return (
      <div className="bg-white rounded-lg p-6 w-full max-w-md">
        <div className="flex justify-between items-center mb-4">
          <h2 className="text-xl font-bold text-gray-800">Tambah Alat Baru</h2>
          <button onClick={onClose} className="text-gray-500 hover:text-gray-700">
            <X size={24} />
          </button>
        </div>
        <div>
          <div className="space-y-4">
            <input
              type="text"
              placeholder="Nama Alat"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.namaAlat}
              onChange={(e) => setFormData({...formData, namaAlat: e.target.value})}
            />
            <input
              type="text"
              placeholder="Type Alat"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.typeAlat}
              onChange={(e) => setFormData({...formData, typeAlat: e.target.value})}
            />
            <input
              type="text"
              placeholder="Serial Number (SN)"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.snAlat}
              onChange={(e) => setFormData({...formData, snAlat: e.target.value})}
            />
            <input
              type="text"
              placeholder="Ruangan"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.ruangan}
              onChange={(e) => setFormData({...formData, ruangan: e.target.value})}
            />
          </div>
          <button
            onClick={handleSubmit}
            className="w-full mt-6 bg-blue-600 text-white py-3 rounded-lg hover:bg-blue-700 font-semibold"
          >
            Simpan
          </button>
        </div>
      </div>
    );
  };

  const KalibrasiForm = ({ equipment, onSubmit, onClose }) => {
    const [formData, setFormData] = useState({
      namaPerusahaan: '',
      laikPakai: 'laik',
      tanggalPengerjaan: '',
      teknisi: ''
    });

    const handleSubmit = () => {
      if (formData.namaPerusahaan && formData.tanggalPengerjaan && formData.teknisi) {
        onSubmit(equipment.id, {
          type: 'kalibrasi',
          ...formData
        });
        onClose();
      }
    };

    return (
      <div className="bg-white rounded-lg p-6 w-full max-w-md">
        <div className="flex justify-between items-center mb-4">
          <h2 className="text-xl font-bold text-gray-800">Kalibrasi</h2>
          <button onClick={onClose} className="text-gray-500 hover:text-gray-700">
            <X size={24} />
          </button>
        </div>
        <div className="mb-4 p-3 bg-gray-100 rounded">
          <p className="text-sm text-gray-600">Alat: <span className="font-semibold">{equipment.namaAlat}</span></p>
          <p className="text-sm text-gray-600">SN: {equipment.snAlat}</p>
        </div>
        <div>
          <div className="space-y-4">
            <input
              type="text"
              placeholder="Nama Perusahaan"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.namaPerusahaan}
              onChange={(e) => setFormData({...formData, namaPerusahaan: e.target.value})}
            />
            <select
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.laikPakai}
              onChange={(e) => setFormData({...formData, laikPakai: e.target.value})}
            >
              <option value="laik">Laik Pakai</option>
              <option value="tidak_laik">Tidak Laik Pakai</option>
            </select>
            <input
              type="date"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.tanggalPengerjaan}
              onChange={(e) => setFormData({...formData, tanggalPengerjaan: e.target.value})}
            />
            <input
              type="text"
              placeholder="Nama Teknisi"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.teknisi}
              onChange={(e) => setFormData({...formData, teknisi: e.target.value})}
            />
          </div>
          <button
            onClick={handleSubmit}
            className="w-full mt-6 bg-green-600 text-white py-3 rounded-lg hover:bg-green-700 font-semibold"
          >
            Simpan Kalibrasi
          </button>
        </div>
      </div>
    );
  };

  const PerbaikanForm = ({ equipment, onSubmit, onClose }) => {
    const [formData, setFormData] = useState({
      namaPerusahaan: '',
      jenis: 'perbaikan',
      penyebab: '',
      teknisi: '',
      sudahDiperbaiki: 'ya',
      troubleshooting: ''
    });

    const handleSubmit = () => {
      if (formData.namaPerusahaan && formData.penyebab && formData.teknisi && formData.troubleshooting) {
        onSubmit(equipment.id, {
          type: 'perbaikan_pemeliharaan',
          ...formData
        });
        onClose();
      }
    };

    return (
      <div className="bg-white rounded-lg p-6 w-full max-w-md max-h-[90vh] overflow-y-auto">
        <div className="flex justify-between items-center mb-4">
          <h2 className="text-xl font-bold text-gray-800">Perbaikan & Pemeliharaan</h2>
          <button onClick={onClose} className="text-gray-500 hover:text-gray-700">
            <X size={24} />
          </button>
        </div>
        <div className="mb-4 p-3 bg-gray-100 rounded">
          <p className="text-sm text-gray-600">Alat: <span className="font-semibold">{equipment.namaAlat}</span></p>
          <p className="text-sm text-gray-600">SN: {equipment.snAlat}</p>
        </div>
        <div>
          <div className="space-y-4">
            <input
              type="text"
              placeholder="Nama Perusahaan"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.namaPerusahaan}
              onChange={(e) => setFormData({...formData, namaPerusahaan: e.target.value})}
            />
            <select
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.jenis}
              onChange={(e) => setFormData({...formData, jenis: e.target.value})}
            >
              <option value="perbaikan">Perbaikan</option>
              <option value="pemeliharaan">Pemeliharaan</option>
            </select>
            <textarea
              placeholder="Penyebab"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              rows="3"
              value={formData.penyebab}
              onChange={(e) => setFormData({...formData, penyebab: e.target.value})}
            />
            <input
              type="text"
              placeholder="Nama Teknisi"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.teknisi}
              onChange={(e) => setFormData({...formData, teknisi: e.target.value})}
            />
            {formData.jenis === 'perbaikan' && (
              <select
                className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                value={formData.sudahDiperbaiki}
                onChange={(e) => setFormData({...formData, sudahDiperbaiki: e.target.value})}
              >
                <option value="ya">Sudah Diperbaiki</option>
                <option value="tidak">Belum Diperbaiki</option>
              </select>
            )}
            <textarea
              placeholder="Troubleshooting"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              rows="3"
              value={formData.troubleshooting}
              onChange={(e) => setFormData({...formData, troubleshooting: e.target.value})}
            />
          </div>
          <button
            onClick={handleSubmit}
            className="w-full mt-6 bg-orange-600 text-white py-3 rounded-lg hover:bg-orange-700 font-semibold"
          >
            Simpan Data
          </button>
        </div>
      </div>
    );
  };

  const LaporanForm = ({ equipment, onSubmit, onClose }) => {
    const [formData, setFormData] = useState({
      namaPelapor: '',
      laporan: ''
    });

    const handleSubmit = () => {
      if (formData.namaPelapor && formData.laporan) {
        onSubmit(equipment.id, {
          type: 'laporan',
          ...formData
        });
        onClose();
      }
    };

    return (
      <div className="bg-white rounded-lg p-6 w-full max-w-md">
        <div className="flex justify-between items-center mb-4">
          <h2 className="text-xl font-bold text-gray-800">Laporan</h2>
          <button onClick={onClose} className="text-gray-500 hover:text-gray-700">
            <X size={24} />
          </button>
        </div>
        <div className="mb-4 p-3 bg-gray-100 rounded">
          <p className="text-sm text-gray-600">Alat: <span className="font-semibold">{equipment.namaAlat}</span></p>
          <p className="text-sm text-gray-600">SN: {equipment.snAlat}</p>
        </div>
        <div>
          <div className="space-y-4">
            <input
              type="text"
              placeholder="Nama Pelapor"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              value={formData.namaPelapor}
              onChange={(e) => setFormData({...formData, namaPelapor: e.target.value})}
            />
            <textarea
              placeholder="Isi Laporan"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
              rows="4"
              value={formData.laporan}
              onChange={(e) => setFormData({...formData, laporan: e.target.value})}
            />
          </div>
          <button
            onClick={handleSubmit}
            className="w-full mt-6 bg-red-600 text-white py-3 rounded-lg hover:bg-red-700 font-semibold"
          >
            Kirim Laporan ke PIC
          </button>
        </div>
      </div>
    );
  };

  const RecordView = ({ equipment, onClose }) => {
    const records = equipment.records || [];
    
    const formatDate = (dateString) => {
      return new Date(dateString).toLocaleDateString('id-ID', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      });
    };

    return (
      <div className="bg-white rounded-lg p-6 w-full max-w-2xl max-h-[90vh] overflow-y-auto">
        <div className="flex justify-between items-center mb-4">
          <h2 className="text-xl font-bold text-gray-800">Record History</h2>
          <button onClick={onClose} className="text-gray-500 hover:text-gray-700">
            <X size={24} />
          </button>
        </div>
        <div className="mb-4 p-3 bg-gray-100 rounded">
          <p className="text-sm text-gray-600">Alat: <span className="font-semibold">{equipment.namaAlat}</span></p>
          <p className="text-sm text-gray-600">SN: {equipment.snAlat}</p>
          <p className="text-sm text-gray-600">Ruangan: {equipment.ruangan}</p>
        </div>
        
        {records.length === 0 ? (
          <p className="text-center text-gray-500 py-8">Belum ada record</p>
        ) : (
          <div className="space-y-4">
            {records.map((record) => (
              <div key={record.id} className="border rounded-lg p-4">
                <div className="flex justify-between items-start mb-2">
                  <span className={`px-3 py-1 rounded-full text-xs font-semibold ${
                    record.type === 'kalibrasi' ? 'bg-green-100 text-green-800' :
                    record.type === 'perbaikan_pemeliharaan' ? 'bg-orange-100 text-orange-800' :
                    'bg-red-100 text-red-800'
                  }`}>
                    {record.type === 'kalibrasi' ? 'Kalibrasi' :
                     record.type === 'perbaikan_pemeliharaan' ? 'Perbaikan/Pemeliharaan' :
                     'Laporan'}
                  </span>
                  <span className="text-xs text-gray-500">{formatDate(record.timestamp)}</span>
                </div>
                
                {record.type === 'kalibrasi' && (
                  <div className="space-y-1 text-sm">
                    <p><span className="font-semibold">Perusahaan:</span> {record.namaPerusahaan}</p>
                    <p><span className="font-semibold">Status:</span> {record.laikPakai === 'laik' ? 'Laik Pakai' : 'Tidak Laik Pakai'}</p>
                    <p><span className="font-semibold">Tanggal:</span> {record.tanggalPengerjaan}</p>
                    <p><span className="font-semibold">Teknisi:</span> {record.teknisi}</p>
                  </div>
                )}
                
                {record.type === 'perbaikan_pemeliharaan' && (
                  <div className="space-y-1 text-sm">
                    <p><span className="font-semibold">Perusahaan:</span> {record.namaPerusahaan}</p>
                    <p><span className="font-semibold">Jenis:</span> {record.jenis === 'perbaikan' ? 'Perbaikan' : 'Pemeliharaan'}</p>
                    <p><span className="font-semibold">Penyebab:</span> {record.penyebab}</p>
                    <p><span className="font-semibold">Teknisi:</span> {record.teknisi}</p>
                    {record.jenis === 'perbaikan' && (
                      <p><span className="font-semibold">Status:</span> {record.sudahDiperbaiki === 'ya' ? 'Sudah Diperbaiki' : 'Belum Diperbaiki'}</p>
                    )}
                    <p><span className="font-semibold">Troubleshooting:</span> {record.troubleshooting}</p>
                  </div>
                )}
                
                {record.type === 'laporan' && (
                  <div className="space-y-1 text-sm">
                    <p><span className="font-semibold">Pelapor:</span> {record.namaPelapor}</p>
                    <p><span className="font-semibold">Laporan:</span> {record.laporan}</p>
                  </div>
                )}
              </div>
            ))}
          </div>
        )}
      </div>
    );
  };

  const QRCodeDisplay = ({ data }) => {
    return (
      <div className="bg-white p-6 rounded-lg border-4 border-gray-800">
        <div className="text-center mb-2">
          <div className="text-6xl">⊞</div>
          <p className="text-xs text-gray-600 mt-2">QR Code: {data}</p>
        </div>
      </div>
    );
  };

  const filteredEquipment = equipment.filter(eq =>
    eq.namaAlat.toLowerCase().includes(searchTerm.toLowerCase()) ||
    eq.snAlat.toLowerCase().includes(searchTerm.toLowerCase()) ||
    eq.ruangan.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100">
      {notification && (
        <div className={`fixed top-4 right-4 z-50 px-6 py-3 rounded-lg shadow-lg flex items-center gap-2 ${
          notification.type === 'success' ? 'bg-green-500' : 'bg-red-500'
        } text-white`}>
          {notification.type === 'success' ? <CheckCircle size={20} /> : <AlertCircle size={20} />}
          {notification.message}
        </div>
      )}

      <div className="container mx-auto px-4 py-8">
        <div className="bg-white rounded-2xl shadow-xl p-6 mb-6">
          <h1 className="text-3xl font-bold text-gray-800 mb-2">Sistem Manajemen Alat Rumah Sakit</h1>
          <p className="text-gray-600">Kelola alat medis dengan sistem QR Code</p>
        </div>

        {view === 'home' && (
          <>
            <div className="flex gap-4 mb-6">
              <button
                onClick={() => {
                  setShowModal(true);
                  setModalType('add');
                }}
                className="flex-1 bg-blue-600 text-white py-4 rounded-xl hover:bg-blue-700 font-semibold flex items-center justify-center gap-2 shadow-lg"
              >
                <Plus size={24} />
                Tambah Alat Baru
              </button>
            </div>

            <div className="mb-6">
              <div className="relative">
                <Search className="absolute left-4 top-1/2 transform -translate-y-1/2 text-gray-400" size={20} />
                <input
                  type="text"
                  placeholder="Cari alat, SN, atau ruangan..."
                  className="w-full pl-12 pr-4 py-3 border rounded-xl focus:ring-2 focus:ring-blue-500 bg-white shadow"
                  value={searchTerm}
                  onChange={(e) => setSearchTerm(e.target.value)}
                />
              </div>
            </div>

            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              {filteredEquipment.map((eq) => (
                <div key={eq.id} className="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow">
                  <div className="flex justify-between items-start mb-4">
                    <div>
                      <h3 className="text-lg font-bold text-gray-800">{eq.namaAlat}</h3>
                      <p className="text-sm text-gray-600">Type: {eq.typeAlat}</p>
                      <p className="text-sm text-gray-600">SN: {eq.snAlat}</p>
                      <p className="text-sm text-gray-600">Ruangan: {eq.ruangan}</p>
                    </div>
                    <QRCodeDisplay data={generateQRData(eq)} />
                  </div>
                  
                  <div className="grid grid-cols-2 gap-2 mt-4">
                    <button
                      onClick={() => {
                        setSelectedEquipment(eq);
                        setModalType('kalibrasi');
                        setShowModal(true);
                      }}
                      className="bg-green-500 text-white py-2 px-3 rounded-lg hover:bg-green-600 text-sm font-semibold"
                    >
                      Kalibrasi
                    </button>
                    <button
                      onClick={() => {
                        setSelectedEquipment(eq);
                        setModalType('perbaikan');
                        setShowModal(true);
                      }}
                      className="bg-orange-500 text-white py-2 px-3 rounded-lg hover:bg-orange-600 text-sm font-semibold"
                    >
                      Perbaikan
                    </button>
                    <button
                      onClick={() => {
                        setSelectedEquipment(eq);
                        setModalType('laporan');
                        setShowModal(true);
                      }}
                      className="bg-red-500 text-white py-2 px-3 rounded-lg hover:bg-red-600 text-sm font-semibold"
                    >
                      Laporan
                    </button>
                    <button
                      onClick={() => {
                        setSelectedEquipment(eq);
                        setModalType('record');
                        setShowModal(true);
                      }}
                      className="bg-blue-500 text-white py-2 px-3 rounded-lg hover:bg-blue-600 text-sm font-semibold"
                    >
                      Record
                    </button>
                  </div>
                  
                  <div className="mt-3 text-xs text-gray-500 text-center">
                    {eq.records?.length || 0} record tersimpan
                  </div>
                </div>
              ))}
            </div>

            {filteredEquipment.length === 0 && (
              <div className="text-center py-16">
                <p className="text-gray-500 text-lg">
                  {equipment.length === 0 ? 'Belum ada alat. Tambahkan alat pertama Anda!' : 'Tidak ada hasil pencarian'}
                </p>
              </div>
            )}
          </>
        )}

        {showModal && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-40">
            {modalType === 'add' && (
              <AddEquipmentForm
                onSubmit={addEquipment}
                onClose={() => setShowModal(false)}
              />
            )}
            {modalType === 'kalibrasi' && selectedEquipment && (
              <KalibrasiForm
                equipment={selectedEquipment}
                onSubmit={addRecord}
                onClose={() => setShowModal(false)}
              />
            )}
            {modalType === 'perbaikan' && selectedEquipment && (
              <PerbaikanForm
                equipment={selectedEquipment}
                onSubmit={addRecord}
                onClose={() => setShowModal(false)}
              />
            )}
            {modalType === 'laporan' && selectedEquipment && (
              <LaporanForm
                equipment={selectedEquipment}
                onSubmit={addRecord}
                onClose={() => setShowModal(false)}
              />
            )}
            {modalType === 'record' && selectedEquipment && (
              <RecordView
                equipment={selectedEquipment}
                onClose={() => setShowModal(false)}
              />
            )}
          </div>
        )}
      </div>
    </div>
  );
};

export default App;
