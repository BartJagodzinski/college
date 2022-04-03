#include <iostream>
#include <cstdlib>
#include <cstdio>
#include <stdlib.h>
#include <vector>
#include <cassert>
#include <ctime>
#include <memory>
#include <string>
#include <algorithm>
#include <functional>

// simple example of "photo gallery" like in smartphone
// photos dont get deleted after gallery is closed
// photos are deleted only after manual deletion
// all operations are on pointers, 1 photo can be in multiple albums, we dont need multiple copies of that image in albums
// only pointers are copied
// all photos all in Album which is root of that photo gallery,
//			Album
//	  |^^^^^^^^^^^^^^^|
// folder1	       *photo1, *photo2, ...
//    |
// *photo1

class Image {
	bool _exist = true;
	std::string _id;
	size_t _size;				// size in KB
	std::string _format;		// .png .jpg ...
	time_t _taken;				// when picture was taken
	bool _sel = false;			// selected
	bool _fav = false;			// favourite (liked)
public:
	Image(std::string id, size_t size, std::string format, time_t taken) :_id(id), _size(size), _format(format), _taken(taken)
	{
		std::cout << "Img " << _id << " constr." << std::endl;
	}

	Image(const Image&) = delete;

	size_t Size() { return _size; }
	std::string Format() { return _format; }
	std::string Id() { return _id; }
	time_t Taken() { return _taken; }
	bool IsFav() { return _fav; }
	bool IsSel() { return _sel; }
	bool Exist() { return _exist; }

	void AddToFav() { _fav = true; }
	void RemoveFromFav() { _fav = false; }
	void Select() { _sel = true; }
	void Unselect() { _sel = false; }
	void NewID(std::string newId) { _id = newId; }
	void Delete() { _exist = false; this->~Image(); }
	~Image() { std::cout << "Img " << _id << " destr." << std::endl; }
};

class Folder {
	bool _exist;
	std::string _name;
	std::vector<Image*> _folder;
public:

	// create empty folder
	Folder(bool t, std::string name) : _exist(t), _name(name)
	{
		std::cout << "Folder " << name << " constr." << std::endl;
		_folder.reserve(0);
	}

	// create folder with adding 1 image
	Folder(bool t, std::string name, Image* img) : _exist(t), _name(name)
	{
		std::cout << "Folder " << name << "constr." << std::endl;
		_folder.reserve(_folder.size() + 1);
		_folder.push_back(img);
	}

	// create folder with vec of images
	Folder(bool t, std::string name, std::vector<Image*> imgs) : _exist(t), _name(name)
	{
		std::cout << "Folder " << name << " constr." << std::endl;
		_folder.reserve(_folder.size() + imgs.size());

		for (Image* i : imgs)
			_folder.push_back(i);
	}

	std::string Name() { return _name; }
	Image* Get(size_t id) { return _folder[id]; }
	size_t Size() { return _folder.size(); }
	std::vector<Image*> GetFolder() { return _folder; }

	void Add(Image* img) { _folder.reserve(_folder.size() + 1); _folder.push_back(img); }

	void Add(std::vector<Image*> imgs) { _folder.reserve(_folder.size() + imgs.size()); for (Image* i : imgs)_folder.push_back(i); }

	void Print() { std::cout << _name << " includes: " << std::endl; for (Image* img : _folder) std::cout << " |___ " << img->Id() << std::endl; }

	void RemoveFromFolder(Image* img) {
		auto it = std::find(_folder.begin(), _folder.end(), img);

		if (it != _folder.end()) _folder.erase(it);
	}

	void DeleteAllImg() { for (Image* img : _folder) { RemoveFromFolder(img); if (img->Exist())img->Delete(); } }

	void Delete() { this->~Folder(); }

	~Folder() { std::cout << "Folder " << _name << " destr." << std::endl; }
};

class Album {
	bool _exist;
	std::vector<Image*> _album;
	std::vector<Folder*> _folders;
	Folder _selected = Folder(true, "selected");

public:
	Album(bool t) { std::cout << "Album constr." << std::endl; _exist = t; _album.reserve(0); _folders.reserve(0); }

	size_t Size() { return _album.size(); }
	std::vector<Image*> GetSelected() { return _selected.GetFolder(); }

	void AddImg(Image* img) { _album.reserve(_album.size() + 1); _album.push_back(img); }

	// add empty folder
	void AddFolder(Folder* folder) { _folders.reserve(_folders.size() + 1); }

	// add folder with adding 1 image
	void AddFolder(Folder* folder, Image* img) {
		_folders.reserve(_folders.size() + 1);
		folder->Add(img);
		_folders.push_back(folder);
	}

	// add folder with vec of images
	void AddFolder(Folder* folder, std::vector<Image*> selected) {
		_folders.reserve(_folders.size() + selected.size());

		for (Image* img : selected)
			folder->Add(img);
		_folders.push_back(folder);
	}

	// add to 1 img to folder
	void AddToFolder(Folder* folder, Image* img) {
		auto it = std::find(_folders.begin(), _folders.end(), folder);
		if (it != _folders.end()) (*it)->Add(img);
	}

	// add to folder vec of images
	void AddToFolder(Folder* folder, std::vector<Image*> selected) {
		auto it = std::find(_folders.begin(), _folders.end(), folder);

		if (it != _folders.end()) (*it)->Add(selected);
	}

	void SelectImg(Image* img) { img->Select();	_selected.Add(img); }

	void UnselectImg(Image* img) { img->Unselect(); _selected.RemoveFromFolder(img); }

	void UnselectAll() { for (Image* img : GetSelected()) UnselectImg(img); }

	void DeleteSelected() {
		for (Image* img : GetSelected()) {
			_selected.RemoveFromFolder(img);
			Del(img);
		}
	}

	void Del(Image* im) {

		for (Folder* f : _folders) {
			f->RemoveFromFolder(im);

			auto it = std::find(_album.begin(), _album.end(), im);
			if (it != _album.end()) _album.erase(it);

			if (im->Exist())	im->Delete();
		}
	}

	void Wipe() {
		char choice = 'n';
		std::cout << "Delete all photos? y/n : ";
		std::cin >> choice;
		if (choice == 'y' || choice == 'Y') { UnselectAll(); for (Folder* folder : _folders) { folder->DeleteAllImg(); folder->Delete(); } this->~Album(); }
		else return;
	}

	~Album() { std::cout << "Album destr." << std::endl; _album.clear(); _folders.clear(); }
};

int main() {
	time_t now = time(0);

	Image* img1 = new Image("photo1", 1024, "png", now);
	Image* img2 = new Image("photo2", 512, "jpg", now + 1);
	Image* img3 = new Image("photo3", 1024, "png", now + 2);
	Image* img4 = new Image("photo4", 512, "jpg", now + 3);
	Image* img5 = new Image("photo5", 1024, "png", now + 4);
	Album* album = new Album(true);
	Folder* folder1 = new Folder(true, "first");
	Folder* old = new Folder(true, "old");

	album->AddImg(img1);
	album->AddImg(img2);
	album->AddImg(img3);
	album->AddImg(img4);
	album->AddImg(img5);

	album->AddFolder(folder1, img1);
	album->AddToFolder(folder1, img2);
	album->AddToFolder(folder1, img3);
	album->AddToFolder(folder1, img4);
	album->AddToFolder(folder1, img5);
	album->AddToFolder(old, img5);

	folder1->Print();
	old->Print();
	// delete img
	album->Del(img1);
	// select image
	album->SelectImg(img3);
	// del selected images
	album->DeleteSelected();
		
	// select more images
	album->SelectImg(img4);
	album->SelectImg(img5);

	// add selected images to album
	album->AddFolder(old, album->GetSelected());
	album->UnselectAll();

	folder1->Print();
	old->Print();

	// img5 is in 2 different folders, select it and delete
	album->SelectImg(img5);
	album->DeleteSelected();

	folder1->Print();
	old->Print();

	// delete everything
	album->Wipe();

	std::cout << "\n\n" << std::endl;
	return EXIT_SUCCESS;
}
