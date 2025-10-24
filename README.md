import React, { useState, useRef } from 'react';
import { Upload, File, Image, FileText, Music, Video, Archive, X, Download, Trash2, Check, AlertCircle } from 'lucide-react';

export default function FileUploadManager() {
  const [files, setFiles] = useState([]);
  const [dragActive, setDragActive] = useState(false);
  const [uploading, setUploading] = useState(false);
  const [notification, setNotification] = useState(null);
  const fileInputRef = useRef(null);

  const getFileIcon = (type) => {
    if (type.startsWith('image/')) return <Image className="text-blue-500" size={24} />;
    if (type.startsWith('video/')) return <Video className="text-purple-500" size={24} />;
    if (type.startsWith('audio/')) return <Music className="text-green-500" size={24} />;
    if (type.includes('pdf')) return <FileText className="text-red-500" size={24} />;
    if (type.includes('zip') || type.includes('rar')) return <Archive className="text-yellow-500" size={24} />;
    return <File className="text-gray-500" size={24} />;
  };

  const formatFileSize = (bytes) => {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i];
  };

  const showNotification = (message, type = 'success') => {
    setNotification({ message, type });
    setTimeout(() => setNotification(null), 3000);
  };

  const handleDrag = (e) => {
    e.preventDefault();
    e.stopPropagation();
    if (e.type === "dragenter" || e.type === "dragover") {
      setDragActive(true);
    } else if (e.type === "dragleave") {
      setDragActive(false);
    }
  };

  const handleDrop = (e) => {
    e.preventDefault();
    e.stopPropagation();
    setDragActive(false);

    const droppedFiles = Array.from(e.dataTransfer.files);
    handleFiles(droppedFiles);
  };

  const handleFileInput = (e) => {
    const selectedFiles = Array.from(e.target.files);
    handleFiles(selectedFiles);
  };

  const handleFiles = (newFiles) => {
    setUploading(true);
    
    const processedFiles = newFiles.map(file => ({
      id: Math.random().toString(36).substring(7),
      name: file.name,
      size: file.size,
      type: file.type,
      uploadDate: new Date().toISOString(),
      progress: 0,
      status: 'uploading'
    }));

    setFiles(prev => [...prev, ...processedFiles]);

    // Simulate upload progress
    processedFiles.forEach((file, index) => {
      let progress = 0;
      const interval = setInterval(() => {
        progress += Math.random() * 30;
        if (progress >= 100) {
          progress = 100;
          clearInterval(interval);
          setFiles(prev => prev.map(f => 
            f.id === file.id ? { ...f, progress: 100, status: 'completed' } : f
          ));
          if (index === processedFiles.length - 1) {
            setUploading(false);
            showNotification(`${processedFiles.length} file(s) uploaded successfully!`);
          }
        } else {
          setFiles(prev => prev.map(f => 
            f.id === file.id ? { ...f, progress: Math.min(progress, 99) } : f
          ));
        }
      }, 200);
    });
  };

  const deleteFile = (id) => {
    setFiles(prev => prev.filter(f => f.id !== id));
    showNotification('File deleted successfully', 'success');
  };

  const downloadFile = (file) => {
    showNotification(`Downloading ${file.name}...`, 'info');
  };

  const totalSize = files.reduce((acc, file) => acc + file.size, 0);
  const completedFiles = files.filter(f => f.status === 'completed').length;

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 via-white to-purple-50 p-4 md:p-8">
      <div className="max-w-6xl mx-auto">
        {/* Header */}
        <div className="mb-8">
          <h1 className="text-4xl font-bold text-gray-800 mb-2">File Upload Manager</h1>
          <p className="text-gray-600">Upload, manage and organize your files effortlessly</p>
        </div>

        {/* Notification */}
        {notification && (
          <div className={`mb-6 p-4 rounded-lg flex items-center gap-3 animate-in slide-in-from-top ${
            notification.type === 'success' ? 'bg-green-100 text-green-800' : 
            notification.type === 'error' ? 'bg-red-100 text-red-800' : 
            'bg-blue-100 text-blue-800'
          }`}>
            {notification.type === 'success' ? <Check size={20} /> : <AlertCircle size={20} />}
            <span className="font-medium">{notification.message}</span>
          </div>
        )}

        {/* Stats */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
          <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-100">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-gray-500 text-sm mb-1">Total Files</p>
                <p className="text-3xl font-bold text-gray-800">{files.length}</p>
              </div>
              <div className="w-12 h-12 bg-blue-100 rounded-lg flex items-center justify-center">
                <File className="text-blue-600" size={24} />
              </div>
            </div>
          </div>

          <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-100">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-gray-500 text-sm mb-1">Completed</p>
                <p className="text-3xl font-bold text-gray-800">{completedFiles}</p>
              </div>
              <div className="w-12 h-12 bg-green-100 rounded-lg flex items-center justify-center">
                <Check className="text-green-600" size={24} />
              </div>
            </div>
          </div>

          <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-100">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-gray-500 text-sm mb-1">Total Size</p>
                <p className="text-3xl font-bold text-gray-800">{formatFileSize(totalSize)}</p>
              </div>
              <div className="w-12 h-12 bg-purple-100 rounded-lg flex items-center justify-center">
                <Archive className="text-purple-600" size={24} />
              </div>
            </div>
          </div>
        </div>

        {/* Upload Area */}
        <div
          className={`relative bg-white rounded-xl shadow-sm border-2 border-dashed transition-all ${
            dragActive ? 'border-blue-500 bg-blue-50' : 'border-gray-300'
          }`}
          onDragEnter={handleDrag}
          onDragLeave={handleDrag}
          onDragOver={handleDrag}
          onDrop={handleDrop}
        >
          <div className="p-12 text-center">
            <div className="w-20 h-20 bg-gradient-to-br from-blue-500 to-purple-500 rounded-full flex items-center justify-center mx-auto mb-4">
              <Upload className="text-white" size={36} />
            </div>
            <h3 className="text-xl font-semibold text-gray-800 mb-2">
              {dragActive ? 'Drop files here' : 'Upload your files'}
            </h3>
            <p className="text-gray-500 mb-6">
              Drag and drop files here, or click to browse
            </p>
            <input
              ref={fileInputRef}
              type="file"
              multiple
              onChange={handleFileInput}
              className="hidden"
            />
            <button
              onClick={() => fileInputRef.current?.click()}
              className="px-6 py-3 bg-gradient-to-r from-blue-500 to-purple-500 text-white rounded-lg font-medium hover:from-blue-600 hover:to-purple-600 transition-all transform hover:scale-105"
            >
              Choose Files
            </button>
            <p className="text-xs text-gray-400 mt-4">
              Supported formats: Images, Documents, Videos, Audio (Max 10MB)
            </p>
          </div>
        </div>

        {/* Files List */}
        {files.length > 0 && (
          <div className="mt-8">
            <h2 className="text-2xl font-bold text-gray-800 mb-4">
              Uploaded Files ({files.length})
            </h2>
            <div className="space-y-3">
              {files.map((file) => (
                <div
                  key={file.id}
                  className="bg-white rounded-xl shadow-sm border border-gray-100 p-4 hover:shadow-md transition-shadow"
                >
                  <div className="flex items-center gap-4">
                    <div className="w-12 h-12 bg-gray-50 rounded-lg flex items-center justify-center flex-shrink-0">
                      {getFileIcon(file.type)}
                    </div>
                    
                    <div className="flex-1 min-w-0">
                      <h3 className="font-semibold text-gray-800 truncate">{file.name}</h3>
                      <div className="flex items-center gap-4 text-sm text-gray-500 mt-1">
                        <span>{formatFileSize(file.size)}</span>
                        <span>•</span>
                        <span>{new Date(file.uploadDate).toLocaleDateString()}</span>
                        {file.status === 'completed' && (
                          <>
                            <span>•</span>
                            <span className="text-green-600 font-medium flex items-center gap-1">
                              <Check size={14} /> Uploaded
                            </span>
                          </>
                        )}
                      </div>
                      
                      {file.status === 'uploading' && (
                        <div className="mt-2">
                          <div className="w-full bg-gray-200 rounded-full h-1.5">
                            <div
                              className="bg-gradient-to-r from-blue-500 to-purple-500 h-1.5 rounded-full transition-all duration-300"
                              style={{ width: `${file.progress}%` }}
                            />
                          </div>
                          <span className="text-xs text-gray-500 mt-1">
                            {Math.round(file.progress)}%
                          </span>
                        </div>
                      )}
                    </div>

                    {file.status === 'completed' && (
                      <div className="flex gap-2">
                        <button
                          onClick={() => downloadFile(file)}
                          className="p-2 hover:bg-blue-50 rounded-lg transition-colors group"
                          title="Download"
                        >
                          <Download size={20} className="text-gray-600 group-hover:text-blue-600" />
                        </button>
                        <button
                          onClick={() => deleteFile(file.id)}
                          className="p-2 hover:bg-red-50 rounded-lg transition-colors group"
                          title="Delete"
                        >
                          <Trash2 size={20} className="text-gray-600 group-hover:text-red-600" />
                        </button>
                      </div>
                    )}
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {files.length === 0 && (
          <div className="text-center py-12">
            <div className="w-32 h-32 bg-gray-100 rounded-full flex items-center justify-center mx-auto mb-4">
              <Upload className="text-gray-400" size={48} />
            </div>
            <p className="text-gray-500 text-lg">No files uploaded yet</p>
            <p className="text-gray-400 text-sm mt-2">Start by uploading your first file</p>
          </div>
        )}
      </div>
    </div>
  );
}
